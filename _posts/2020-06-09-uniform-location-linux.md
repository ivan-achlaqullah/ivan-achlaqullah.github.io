---
title: "Ketika OpenGL Lebih Lemot di Linux"
categories:
  - Blog
tags:
  - buri engine
  - opengl
  - linux
  - amd
---

Salah satu manfaat `porting` aplikasi dari Windows ke Linux adalah menemukan beberapa bug yang tidak disadari.

Waktu itu ketika lagi porting [Buri Engine](https://github.com/ivan-achlaqullah/buri-engine/) ke Linux sempat kaget gara-gara lemot sekali rendernya. Biasanya pada `Debug` bisa 90 FPS ke atas di windows ini jadi cuma 15 FPS di linux. Seharusnya OpenGL setara atau lebih cepat di linux dari pada di windows, jadi pasti ada yang tidak efisien di dalam kodenya.

Setelah [ditelusuri](https://github.com/ivan-achlaqullah/buri-engine/commit/8d83f57e21241a94b4499f3a33bc3d6d46ef6363) akhirnya ketemu juga penyebabnya. Tetapi sebelum membahas penyebabnya lebih baik amati dulu kode di bawah ini:

```c++
// Send Model Matrix Uniform
auto location = glGetUniformLocation(programIndex, "model");
glUniformMatrix4fv(location, 1, GL_FALSE, glm::value_ptr(transform->_modelMatrix));

// Send Material Data
auto material = ecs::get_component<component::material>(mesh->entityNumber);
location = glGetUniformLocation(programIndex, "ActiveMaterial"); // Slow on linux (AMDGPU RX 480)
glUniform1i(location, material->index_uniform);
```

Oke mari kita jelaskan satu-satu. Kode di atas digunakan untuk mengirim data dari program ke shader yang ada di GPU, dalam hal ini `glm::mat4 _modelMatrix` dan `int index_uniform`. Untuk mengirim data tersebut, kita harus mengetahui "slot" data tersebut di GPU dengan menggunakan `glGetUniformLocation()`.

Setelah mendapatkan lokasinya, data tersebut baru bisa dikirim ke GPU. Akan tetapi fungsi yang digunakan tergantung dari jenis datanya misalnya untuk mengirim `glm::mat4` perlu menggunakan `glUniformMatrix4fv()`, sementara `int` menggunakan `glUniform1i()`.

Masalahnya di Linux (paling gak di Linux yang menggunakan GPU dari AMD) menggunakan `glGetUniformLocation()` untuk mendapatkan lokasi uniform bertipe `int` itu sangat lemot, sementara untuk tipe lain seperti `mat4` dan `vec3` tidak ada masalah sama sekali. Entah apa alasannya, mungkin ada bug di driver AMD di Linux? Sayangnya sangat sulit untuk memperbaiki bug di driver. Untungnya kita bisa "menghindari" permasalahan tersebut tanpa menyentuh driver sama sekali.

Lalu bagaimana solusinya? Dari pada memanggil `glGetUniformLocation()` setiap frame, kita memanggilnya **di awal program** saja lalu menyimpan lokasinya. Sehingga lemotnya hanya di awal program saja, tidak di setiap frame.

```c++
// Sebelum
auto material = ecs::get_component<component::material>(mesh->entityNumber);
location = glGetUniformLocation(programIndex, "ActiveMaterial");
glUniform1i(location, material->index_uniform);

// Sesudah
auto material = ecs::get_component<component::material>(mesh->entityNumber);
glUniform1i(s_shaderBank[mesh->shaderProgram]._location_active_material, material->index_uniform);
```

Setelah melakukan hal tersebut, FPS yang didapatkan di Linux setara dengan di Windows.