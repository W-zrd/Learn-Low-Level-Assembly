# Learn-Low-Level-Assembly

# Description

**My Notion notes when learning about the basics of Assembly, Computer Architecture, and a bit of Operating Systems** started from early-2023. This repo is basically my learning progress about assembly which will be updated continuously whenever I get confused.

I have an interest in low-level analysis, such as reversing a program, Return Oriented Programming, and exploiting ELF binary, which is why im learning this language. I often forget how things work in Assembly and Computer Architecture because it’s so hard to understand at the beginning. So when it happens, i usually take a note. And here's what I've learned so far.

# Table of Contents

- **Memory Layout - (TODO)**
- [**Stack Layout**](#stack-layout)
    - [Arsitektur x86](#arsitektur-x86)
    - [Arsitektur x86-64](#arsitektur-x86-64)
- [**Register**](#)
    - [General Purpose Register](#general-purpose-register)
    - [Index Register](#index-register)
    - [Pointer Register](#pointer-register)
- [Memory Address](#memory-address)
    - [Byte Ordering (Endianness)](#byte-ordering-endianness)
        - [Most Significant Bit (MSB)](#most-significant-bit-msb)
        - [Least Significant Bit (LSB)](#least-significant-bit-lsb)
        - [Big Endian](#big-endian)
        - [Little Endian](#little-endian)
    - [Base Address & Offset](#base-address--offset---todo)
- [System Call](#system-call)
    - [`sys_open`](#sys_open)
    - [`sys_read`](#sys_read)
    - [`sys_write`](#sys_write)
    - [`sys_exit`](#sys_exit)
- [Addressing Modes](#addressing-modes)
- [Practice (Reversing a simple program) - (TODO)](#practice-reversing-a-simple-program---todo)

# Stack Layout

Stack bersifat LIFO (Last In, First Out), yang berarti data yang dimasukkan terakhir akan keluar pertama kali. **Stack tumbuh dari address yang tinggi ke address yang lebih rendah**. Namun, susunan variable local pada stack di memory tergantung compilernya

Stack biasanya digunakan untuk menyimpan data yang bersifat sementara, dan ketika fungsi selesai dieksekusi, frame stacknya dihapus dari stack. Hal ini berbeda dengan heap, yang digunakan untuk alokasi memori dinamis yang dapat bertahan lebih lama.

Setiap arsitektur prosesor memiliki layout yang berbeda. 

## Arsitektur x86

Seperti yang sudah dijelaskan sebelumnya, di dalam layout memory terdapat Stack yang berfungsi untuk mengelola function call dan return address. Stack ini sendiri terdiri atas beberapa bagian, yaitu sebagai berikut.

![Untitled](Learn-Low-Level-Assembly%20ad401c8d8b9c4bdc9534d44777832ba2/Untitled.png)

- **Saved EBP:** Nilai EBP yang disimpan pada awal frame. Lokasi awal (base) dari frame fungsi saat ini
- **Return Address:** Saat fungsi dipanggil, return address disimpan di dalam stack agar program dapat kembali ke instruksi berikutnya setelah function call selesai

Data dimasukkan ke dalam stack menggunakan operasi "push", sedangkan dihapus menggunakan operasi "pop". Parameter fungsi akan dipush ke dalam stack mulai dari parameter terakhir sampai parameter pertama. 

## Arsitektur x86-64

Terdapat beberapa perbedaan antara arsitektur x86 dengan x86-64.

- **Register:** Pada arsitektur x86-64, nama register diawali dengan prefix “R” (Register). Contohnya seperti RAX, RBX, RCX, dan RDX.
- **Parameter:** Setiap parameter fungsi akan ditambahkan ke dalam register RDI, RSI, RDX, RCX, R8, dan R9 secara berurutan. Jika parameter lebih dari 6, maka akan disimpan di dalam stack.

![Untitled](Learn-Low-Level-Assembly%20ad401c8d8b9c4bdc9534d44777832ba2/Untitled%201.png)

# Register

Register dan stack merupakan 2 konsep yang berbeda.  **Register terletak di dalam prosesor, sedangkan stack terletak di dalam memori**. Oleh karena itu register mampu menyimpan data untuk processing dengan cepat karena terletak di dalam CPU. Data pada register diakses secara langsung oleh prosesor tanpa perlu alamat memori.

Nama register berbeda-beda tergantung arsitekturnya. Misalnya **`RBX`** merupakan register 64 bit, sedangkan **`EBX`** adalah versi 32-bit dari register `RBX`, begitupun dengan register lainnya. Pada arsitektur x86-64 terdapat register tambahan, yaitu `r8` sampai `r15`. Register tersebut hanya ada pada arsitektur x86-64 saja.

![Untitled](Learn-Low-Level-Assembly%20ad401c8d8b9c4bdc9534d44777832ba2/Untitled%202.png)

Note: Higher 8-bit register pada Index dan Pointer register tidak dapat diakses pada memori.

### General Purpose Register

- **Accumulator (RAX):** menyimpan hasil dari operasi aritmetika dan logika, misal: return address dari fungsi
- **Base Register (RBX):** menyimpan alamat awal dari suatu data/array
- **Counter Register (RCX):** sering digunakan pada loop atau operasi yang memerlukan iterasi berulang.
- **Data Register (RDX):** register data tambahan dalam beberapa operasi aritmetika fatau logika.

### Index Register

- **Destination Index (RDI):** Biasa digunakan untuk menyimpan parameter pertama pada suatu fungsi
- **Source Index (RSI):** Biasa digunakan untuk menyimpan parameter kedua pada suatu fungsi

### Pointer Register

- **Base Pointer (RBP):** Pointer untuk mengakses variabel lokal dan parameter fungsi, menunjuk ke awal dari suatu stack frame
- **Stack Pointer (RSP):** Pointer yang menunjuk ke alamat top of stack, dan Menyimpan alamat memori dari stack.
- **Instruction Pointer (RIP):** Menyimpan memory address dari instruksi yang akan dieksekusi sehingga prosesor tahu instruksi mana yang harus dijalankan selanjutnya.

# Memory Address

Alamat memori selalu direpresentasikan dalam bentuk heksadesimal. Dalam sistem bilangan heksadesimal, setiap digit mewakili 4 bit (setengah byte) data. **Alamat memori pada sistem operasi dapat menyimpan 1 byte data**. Oleh karena itu untuk mewakili 1 byte dalam notasi hexa, kita memerlukan dua digit hexadecimal.

![Untitled](Learn-Low-Level-Assembly%20ad401c8d8b9c4bdc9534d44777832ba2/Untitled%203.png)

Dalam sistem 32-bit, sebuah alamat memori memiliki panjang 32 bit (4 byte). Jika kita ingin merepresentasikan alamat memori itu dalam notasi hex, kita membutuhkan 8 digit hexa karena setiap digit mewakili 4 bit. Jadi, untuk memecah alamat memori 32-bit menjadi empat bagian, masing-masing direpresentasikan oleh dua digit heksadesimal, kita memperoleh total 8 digit heksadesimal.

## Byte Ordering (*Endianness*)

Endianness adalah cara memori dalam mengurutkan address pada memori komputer. Data disusun dalam memori berdasarkan sistem endianness. LSB (*Least Significant Bit*) dan MSB *(Most Significant Bit*) adalah istilah yang digunakan dalam konteks representasi biner dari data.

### *Most Significant Bit* (MSB)

- MSB adalah bit yang memiliki nilai paling besar atau paling signifikan dalam sebuah bilangan
- Misalnya pada representasi desimal, bilangan yang paling signifikan adalah digit pertama atau digit yang paling kiri.
- Contoh: Dalam bilangan biner 10100, bit paling kiri (atau pertama) adalah MSB, yaitu nilai 1.

### *Least Significant Bit* (LSB)

- LSB adalah bit yang memiliki nilai paling kecil atau paling sedikit bobot/valuenya dalam suatu bilangan
- Misalnya pada representasi desimal, bilangan yang paling kecil adalah digit terakhir atau paling kanan.
- Contoh: Dalam bilangan biner 10100, bit paling kanan (atau terakhir) adalah LSB, yang memiliki nilai 0.

![Untitled](Learn-Low-Level-Assembly%20ad401c8d8b9c4bdc9534d44777832ba2/Untitled%204.png)

Pada dunia komputer, ada dua jenis endianness:

### 1. Big Endian

- *Most Significant Bit* (MSB) ditempatkan di alamat memori paling rendah (atau alamat yang lebih kecil).
- Nilai yang paling penting dalam representasi bilangan ditempatkan di paling kiri.
- **Contoh:** IBM PowerPC dan Motorola 68000 series

### 2. Little Endian

- *Least Significant Bit* (LSB) ditempatkan di alamat memori paling rendah (atau alamat yang lebih kecil).
- Bilangan yang ditempatkan di paling kiri adalah nilai yang paling kecil.
- **Contoh:** Intel x86, ARM, AMD, dan MIPS

## Base Address & Offset

- **Base address:** Alamat memori di mana suatu fungsi atau bagian dari program dimulai, atau dengan kata lain alamat memori yang menjadi titik awal untuk suatu area memori tertentu.
- **Offset:** Jarak antara alamat instruksi dengan base address dari suatu fungsi. Offset menunjukkan seberapa jauh alamat instruksi tersebut berada dari awal blok kode.

Jadi, base address adalah titik awal atau referensi, sedangkan offset adalah jarak atau pergeseran dari titik tersebut. Offset dapat dihitung dengan rumus :

> Offset = AlamatInstruksi - Base Address

# Addressing Modes

![Untitled](Learn-Low-Level-Assembly%20ad401c8d8b9c4bdc9534d44777832ba2/Untitled%205.png)

### Immediate Addressing

- Nilai operand diberikan secara langsung di dalam instruksi.
- Nilai operand sudah tersedia dan tidak perlu diambil dari memori atau register lain.
- Source Operand adalah sebuah nilai konstan

```nasm
MOV rax, 1
ADD rdi, 1
```

### Register Addressing

- Nilai operand adalah isi dari register tertentu.

```nasm
MOV eax, ebx
ADD ecx, edx
```

### Direct Addressing

- Alamat memori yang dituju (destination operand) diberikan secara langsung dalam instruksi.
- Source operand berisi alamat memori yang langsung ditentukan dalam instruksi.

```nasm
section .data
    ebx dd 42

section .text
    global _start

_start:
    mov eax, [ebx]
```

- Kurung siku **`[]`** menandakan pengambilan nilai dari alamat memori yang ditunjukkan di dalam kurung tersebut.
- Pada kode tersebut, **`mov eax, [ebx]`**, register **`eax`** akan berisi nilai yang terdapat di alamat memori yang ditunjukkan oleh nilai yang tersimpan di dalam register **`ebx`**, bukan alamat memori itu sendiri.
- **Tanpa tanda kurung siku**, **`ebx`** **akan dianggap sebagai nilai langsung** yang ingin dimuat ke dalam **`eax`**, bukan sebagai alamat memori yang mengandung nilai yang ingin dimuat.
- Pada kode tersebut, `eax` akan bernilai `42`

### Indirect Addressing

- Alamat memori yang akan diakses tidak diberikan secara langsung dalam instruksi, tetapi diambil dari isi register
- Source operand adalah register yang berisi alamat memori yang akan diakses.

```nasm
section .data
    my_var dd 42

section .text
    global _start

_start:
    mov ebx, my_var
		mov eax, [ebx]
```

### Relative Addressing

- Alamat memori yang digunakan sebagai operand tidak diberikan secara langsung, tetapi dihitung berdasarkan lokasi instruksi saat itu.
- Sering digunakan dalam instruksi `jmp`.

```nasm
1:  MOV eax, 10
2:  JMP 5
3:  ADD eax, 5
4:  ...
5:  LABEL:
6:  ...
```

- Misalnya, ketika program memiliki instruksi **`JMP <label>`**, RIP perlu tahu ke mana harus melompat. Tapi bagaimana RIP mengetahui di mana **`label`** berada? Jawabannya adalah dengan menggunakan relative addressing.

# **Practice (Reversing a simple program) - (TODO)**

## Microcorruption: New Orleans

- **Challenge URL:**
- **Goals:**

## Crackmes: …

- **Challenge URL:**
- **Goals:**