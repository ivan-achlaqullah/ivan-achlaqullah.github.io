---
title: "Apa itu SIMD ?"
categories:
  - Blog
tags:
  - programming
  - optimization
  - simd
  - c++
---

**Warning:** post ini masih belum selesai ditulis, jadi masih ada beberapa bagian yang belum lengkap.
{: .notice--warning}

SIMD (Single Instruction Multiple Data) merupakan salah satu cara untuk mempercepat program dengan cara paralel, *tanpa menggunakan multi-threading sama sekali.* Loh kok bisa? Bukannya kalau mau paralel harus pakai multi-threading?

Anggap saja ada code yang cukup kompleks seperti ini:

```c++
#include <iostream>
#include <array>
#include <chrono>

struct Vec4 {
  union { float x; float a; };
  union { float y; float b; };
  union { float z; float c; };
  union { float w; float d; };
};

struct Mat4 {
  Vec4 a = { 1.f, 0.f, 0.f, 0.f };
  Vec4 b = { 0.f, 1.f, 0.f, 0.f };
  Vec4 c = { 0.f, 0.f, 1.f, 0.f };
  Vec4 d = { 0.f, 0.f, 0.f, 1.f };

  Mat4 operator*(const Mat4& right) {
    //Todo, anggap saja sudah ada kodenya
    Mat4 hasil;
    return hasil;
  }
};

int main() {
  std::array<Mat4, 4000> position_matrix;
  std::array<Mat4, 4000> rotation_matrix;
  std::array<Mat4, 4000> scale_matrix;

  std::array<Mat4, 4000> model_matrix;

  auto start_time = std::chrono::high_resolution_clock::now();
  for (int i = 0; i < 4000; i++) {
    model_matrix[i] = scale_matrix[i] * rotation_matrix[i] * position_matrix[i];
  }
  auto end_time = std::chrono::high_resolution_clock::now();

  float delta = std::chrono::duration<float, std::chrono::milliseconds::period>
    (end_time - start_time).count();

  std::cout << "Durasi : " << delta << " ms\n";
}
```

Jika ditelaah dengan menggunakan profiler, maka bagian paling lambat di dalam kode tersebut adalah `Mat4 operator*`. Kenapa? Karena jika dilihat kode assembly yang dihasilkan...

**Warning:** post ini masih belum selesai ditulis, jadi masih ada beberapa bagian yang belum lengkap.
{: .notice--warning}