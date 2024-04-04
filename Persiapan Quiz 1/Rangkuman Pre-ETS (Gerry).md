# Table of Content
- [Table of Content](#table-of-content)
- [PPT 1](#ppt-1)
  - [Basic Element](#basic-element)
  - [Process Register](#process-register)
    - [User-visible register](#user-visible-register)
    - [Control and Status Registers](#control-and-status-registers)
  - [Instruction Execution](#instruction-execution)
  - [Interupts](#interupts)
    - [4 Jenis Interrupt](#4-jenis-interrupt)
  - [Instruction Execution/Flow with Interrupt](#instruction-executionflow-with-interrupt)
  - [Interrupt Processing](#interrupt-processing)
    - [Simple Interrupt Processing](#simple-interrupt-processing)
    - [Sequential Interrupt Programming](#sequential-interrupt-programming)
    - [Nested Interrupt Programming](#nested-interrupt-programming)
  - [Multiple Interrupt](#multiple-interrupt)
    - [Prioritas](#prioritas)
  - [Multiprogramming](#multiprogramming)
  - [Memory Hierarchy](#memory-hierarchy)
  - [Struktur Cache dan Main Memory](#struktur-cache-dan-main-memory)
  - [Prinsip Cache](#prinsip-cache)
    - [Cara Cache Beroperasi](#cara-cache-beroperasi)
  - [Teknik Komunikasi I/O](#teknik-komunikasi-io)
    - [Programmed I/O](#programmed-io)
    - [Interrupt-Driven I/O](#interrupt-driven-io)
  - [Footnote](#footnote)
    - [Spatial Locality](#spatial-locality)
    - [temporal locality](#temporal-locality)
# PPT 1
## Basic Element 
- Processor
  - dua register di dalam processor
    - Memory Access Register (MAR)
      - ngandung address dari buat read-write berikutnya
    - Memory Buffer Register (MBR)
      - ngandung data yang dibaca atau data yang akan ditu969
- I/O Module
  - dihitung sebagai secondary memory device
  - berupa terminal (ports dan USB stuff)
  - peralatan komunikasi (Keyboard + Mouse, etc)
- System Bus
  - yang membuat main memory, I/O module, dan processor bisa berkomunikasi dengan satu sama lain
## Process Register
Ada dua: 
- User-visible register
  - yang dipake ama programmer misal: accumulator, index register, stack pointer, dll 
    - accumulator itu yang simpen data sementara sebelum di assign ke sebuah register/variabel (cth : a = a+b, instruction bikin a+b disimpen di accumulator dulu, baru write accumulator value to a)
    - index register itu buat nge access/traverse array
    - stack pointer itu buat nge access stack/heap memory
- Control and Status register
  - Dikontrol ama processor biar processor bisa lakuin processor things
  - dipake ama OS buat kontrol eksekusi program (kek cronjob) 
### User-visible register
- Inget kan register itu basically cuma container buat data/variabel, jadi memang semua program bisa declare variabel dan variabel itu pake user-visible register
- jadi user-visible register itu bisa diakses ama user/program yang dibikin ama user
- User-visible register dibagi jadi dua
  - data register
    - buat simpen data literal value
  - address register
    - buat simpen data address
### Control and Status Registers
- Program Counter (PC)
  - mengandung address dari instruksi yang pengen difetch
- Instruction Register (IR)
  - ngandung instruksi yang di fetch ama PC
- Program status word (PSW)
  - ngandung informasi status: interrupt enable/disable, mode, dll
- Condition codes or flags
  - bit yg kasi tahu hasil operasi
    - positif, negative, nol, ato overflow
## Instruction Execution
- 2 Step
  1. Fetch
    - PC pegang address dari instruksi berikutnya
    - ambil instruksi dari memory
    - setiap fetch, PC++ (increment)
  2. Execute
    - instruction yg di fetch masuk ke instruction register
      - (instruction bisa berupa banyak jenis) :
        - instruction yg involve processor-memory
        - instruction yg involve processor-I/O
        - instruction yg involve data processing
        - instruction yg involve control (?)
    - eksekusi instruksi
  3. Loop or Halt
    - loop klo ada instruksri berikutnya
    - halt klo udah selesai
<br/><br/>
![Instruction Execution](./resources/simple-instruction-cycle.png)
## Interupts
- interupt itu cuma signal berhenti (SIGINT) sementara yang dipake buat ngasi tau processor klo ada event yang lebih penting
- Fact: I/O itu mayoritas lebih pelan kerjanya dibanding ama processor
### 4 Jenis Interrupt
  - dari program :
    - terjadi klo programnya sendiri ada error (kek klo di cp (ARYA) itu tle to infinity jadi programnya sendiri interupt gara" gk bakal berhenti) (SIGINT jg bisa terjadi klo programnya sendiri jg error, kek memory overflow ato division by zero)
  - dari timer :
    - pre-planned ama processor klo programnya bakal di interrupt ama processornya expect buat programnya ke interrupt
    - ini exist klo OS nya mo operate pada interval tertentu 
    - cth: sebuah system clock nge hitung waktu per detik, klo sudah menyampai value tertentu, OS nya bakal interrupt clock nya untuk melakukan program yang lain
  - dari I/O
    - klo I/O nya kasi signal klo programnya sudah selesai (return 0) ato request dari user buat stop program nya (ctrl+c dari terminal)
    - note I/O itu gk neccesarily mean input output dari mouse ato keyboard ato dll. Bisa aja itu dari program output return  jadi langsung interrupt
    - bedanya interrupt ini ama interrupt dari program adalah klo interrupt dari program itu karena error yg terjadi di dalam programnya dan gk di expect oleh processor. klo interrupt dari I/O itu karena programnya udah selesai (walaupun return value nya bkn 0 (error), itu masih di return (end program) dan bisa dibilang programnya selesai)
  - dari Hardware Failure
    - contoh : processor pengen ke ambil data dari secondary memory (disk), tapi disk nya rusak jadi gk bisa ke fetch datanya, sehingga processor nya bakal interrupt
- Selama program berjalan, ia akan terus nge cek klo nge catch sebuah interrupt (if interrupt)
- klo ada interrupt, program bisa lakuin dua hal:
  - [suspend](#suspend) program
  - jalanin catch interrupt routine / nge call interupt handler(kek catch nya di try throw catch di c++)
    - Contoh yang lumayan complex : ada process yg request I/O operation, tapi I/O operation kan gk secepat/lebih lama dari processor jadi process nya sendiri masuk ke dalam wait/suspend state, nah pas I/O operation nya selesai, I/O module nya kasi signal ke processor klo I/O operation nya udah selesai, dari situ processor kirim SIGINT (signal interrupt) ke process yg lagi dlm wait status yg kirim pesan bahwa I/O udh siap.
## Instruction Execution/Flow with Interrupt
1. fetch instruction
2. execute instruction
   - interrupt enabled 
     - check interrupt
     - if interrupt, handle interrupt
     - back to fetch
   - interrupt disabled
     - back to fetch
3. loop from fetch, unless process done (halt)
<br/><br/>
![Instruction Execution/Flow with Interrupt](./resources/instruction-cycle-with-interrupt.png)
## Interrupt Processing
### Simple Interrupt Processing
note: ini gw copas dari ppt nya, `jadi mungkin ada beberapa kata yang gk gw pahami`
lol ini copilot yg bikin (i fucking love copilot)
1. Hardware side
   1. hardware demand sebuah interrupt
   2. processor selesaiin instruksi
   3. processor bilang ke hardware (i think?) klo dapat signal interrupt
   4. processor masukin PSW (processor status word) (variabel yg contain data status process (lagi jalan kah, wait kah, suspend kah?, dll) ) dan PC ke dalam control stack (stack yg buat nyimpen instruction sementara)>
   5. processor bikin PC baru buat interrupt handling
2. Software side
   1. simpen kondisi informasi yg ada di dlm process
   2. interupt process
   3. kembaliin kondisi informasi
   4. kembaliin PSW ama PC yg ada di control stack sebelumnya
### Sequential Interrupt Programming
1. Process beroperasi
2. Interruption muncul
3. Pertama dihandle oleh `Interruption Handler X`
4. IHX (Interruption Handler X) return/selesai handle interrupt
5. Selesai IHX, dilanjutkan oleh `Interruption Handler Y` (mereka bekerja pas after one another, gaada break)
6. IHY return/selesai handle interrupt
7. Selesai IHY, process berjalan lagi
<br/><br/>
![Sequential Interrupt Programming](./resources/sequential-interrupt_handle.png)
### Nested Interrupt Programming
1. Process beroperasi
2. Interruption muncul
3. Pertama dihandle oleh `Interruption Handler X`
4. Di dalam IHX ada interrupt lagi yang di handle oleh `Interruption Handler Y`
5. IHY selesai, kembali ke IHX
6. IHX selesai, kembali ke process
7. process berjalan lagi
<br/><br/>
![Nested Interrupt Programming](./resources/nested-interrupt-handle.png)
## Multiple Interrupt
### Prioritas
Di dalam contoh berikut ini, kita pakai kprioritas buat jelasin hierarki kepentingan interrupt
cuma perlu di note aja klo prioritas itu angka yg lebih kecil
<br/>
misalkan:
- process A (pA) memiliki interrupt prioritas 5
- process B (pB) memiliki interrupt prioritas 4
- process C (pC) memiliki interrupt prioritas 2

1. pA, pB, pC beroperasi
2. pA muncul interrupt prioritas 5
3. pA dihandle oleh IHX (Interruption Handler X)
4. Selama IHX lagi jalan, pB muncul interrupt prioritas 4
5. karena prioritas pB lebih tinggi dari pA, IHX di suspend dan pB dihandle oleh IHY terlebih dahulu
6. Selama IHY lagi jalan, pC muncul interrupt prioritas 2
7. karena prioritas pC lebih tinggi dari pB, IHY di suspend dan pC dihandle oleh IHZ terlebih dahulu
8. IHZ selesai, jadi IHY di unsuspended
9. IHY selesai, jadi IHX di unsuspended
10. IHX selesai, process berjalan lagi

## Multiprogramming
- Multiprogramming itu konsep processor memiliki lebih dari 1 program buat di execute sekaligus
- Sequence/urutan dari program mana yang harus diexecute itu berdasarkan [Prioritas](#prioritas) relatif mereka dan berdasarkan apakah mereka menggunakan I/O atau tidak
  - klo program A menggunakan I/O, dan program B gk, maka program B akan di execute terlebih dahulu
- Dalam multiprogramming, klo sebuah interrupt handler selesai, bisa aja handler nya return ke program lain (maksud program lain itu program yg bkn lagi di execute atau program yang bkn merupakan sumber interrupt)

## Memory Hierarchy
Memory pada dasarnya memiliki 3 atribut yg ngefek performance
- capacity
  - se brp banyak data 1 memory block bisa simpen
- access time
  - seberapa cepat data bisa di akses dari memory block
- cost
  - sebuah data di dalam sebuah memory block itu perlu brp banyak bit buat di simpen

Dari atribut ini, kita cuma bisa pilih min-max options :
1. Faster access time, greater cost per bit
  - contoh: Cache (capacity kecil, access time cepat, cost mahal)
2. Greater capacity, smaller cost per bit
  - contoh: RAM (capacity besar, access time relatif lambat, cost murah)
3. Greater capacity, slower access speed
  - contoh: HDD (hard disk) (kapasitas besar banget, access time lambat banget, cost murah)

![Memory Hierarchy](./resources/memory-hierarchy.png)
dari gambar diatas itu klo diperhatiin :
1. semakin ke bawah, cost per bit semakin murah
2. semakin ke bawah, capacity lebih besar
3. semakin ke bawah, access time nya lebih lama
4. Klo diperhatiin, itu jg semakin ke bawah, data-data yang biasanya di bawah hiearki itu lebih jarang dipanggil/kepake ama processor dibanding ama data-data di cache/atas hiearki
<br/><br/>
Bisa dibayangin di atas semua hiearki ini, ada processor dan processor panggil data di memory dari atas, jadi klo cache yg paling deket ama processor, bisa disimpulkan bahwa capacity nya kecil, access time nya cepet banget, dan cost nya mahal.

## Struktur Cache dan Main Memory
![Struktur cache](./resources/cache-structure.png)
Tag merupakan semacam label/prefix untuk instant query block nya. jadi ketika processor mencari sebuah data, processor mencari dari tag
  - klo ketemu (tag hit), maka cache cuma perlu return block dari tag nya
  - klo gk ketemu (tag miss), maka cache jg perlu nge call lagi ke main memory buat ambil block nya tapi tag nya disimpe ke dlm cache jadi next time selalu tag hit.

![Struktur main memory](./resources/main-memory-structure.png)
main memory intinya itu setor words (data) ke dalam block (memory block) dan setiap block itu punya address nya sendiri yg unique. jadi ketika processor nge call cache yg ngne call main memory untuk akses data, processor nge akses block nya bukan langsung data nya

## Prinsip Cache
- Cache mengandung sebuah porsi/sebagian dari main memory
- tiap kali processor mo akses memory, processor nge cek cache terlebih dahulu
  - klo ketemu, gk perlu call ke main memory
  - klo gk ketemu, data dipanggil dari main memory ke cache
    - data yang barusan dipanggil akan disimpen ke cache dan untuk future use case jadi gk perlu lagi panggil dari main memory
  - pokoknya after dipanggil data nya, pasti data reference nya ada di dalam cache after operation call nya
- cache sekecil-kecilnya, itu impactful ama penting banget buat performance
- jumlah data/word yg ke tuker diantara main memory ama cache itu depends entirely ke block size
  - block size yg lebih besar bisa bikin tag hit lebih sering terjadi sampai probabilitas
    - akan tetapi semakin besar size nya --> semakin banyak word yg masuk ke dalam cache --> semakin besar kemungkinan klo semua word yg di dlm block itu gk pake (redundant word yg masuk ke dalam cache) (perlu diinget tapi ada nya prinsip [spatial locality](#spatial-locality) dan [temporal locality](#temporal-locality))
- penentuan cache bakal setor block dmn
  - ketika sebuah block dibaca ke dalam cache, block lain harus di replace (inget block size itu universal, semua block size nya sama), harus ada algoritma buat nge cek block mana yg bakal dipake in the near future
  - semakin flexible algoritma mapping (alokasi block di address), semakin complex block-searching algorithm yg perlu dibikin
- block-replacement algorithm
  - Least-recently-used (LRU) algorithm
  - First in first out (FIFO) algorithm
  - least frequently used (LFU) algorithm
- prinsip cache untuk write block
  - bisa terjadi tiap kali ada block di cache yg di updated
  - bisa terjadi tiap kali block di dalam cache itu di replace
  -  block yg di update di dalam cache itu gk di sync ke main memory immediately until block nya di replace
### Cara Cache Beroperasi
1. Processor minta sebuah word dari cache
2. Cache cek apakah word nya ada di cache nya sendiri
   - klo ada, cache nge fetch word dari storage sendiri, lanjut ke 6.
   - klo gaada, cache nge call main memory untuk fetch wordnya, lanjut ke 3.
3. cache allocate space buat masukin word ke dalam cache nya
4. main memory kasi block of word ke cache
5. cache masukin block nya ke dalam storage 
6. cache return word nya ke processor

## Teknik Komunikasi I/O
### Programmed I/O
Note: kelemahan dari metode ini itu processor harus nge loop terus menerus buat nge cek apakah I/O module nya udah siap atau belum + nge cek apakah operasi I/O nya udah selesai atau belum. Jadi processor kerja ngerjain task yang kurang kerjaan.

1. Processor nge call I/O module
2. baca status I/O module
   - klo I/O module blom siap, nge loop check ampe udh siap
3. pas I/O udh siap, baca word dari I/O module.
4. write word ke main memory.
5. cek apakah operasi semuanya udh selesai?
   - klo blom, loop ke 2

### Interrupt-Driven I/O
1. I/O module kasi signal (interrupt) ke processor klo I/O module nya udah siap (processor kerjain yg lain jadi gk cek I/O module udh siap blom)
2. check status abis dari interrupt (signal) apakah I/O module sesungguhnya sudah siap
   - klo ternyata blom siap tapi I/O module sudah kirim interrupt tadi, error condition
3. processor nge call baca dari I/O module
4. write word ke main memory
5. processor nge ceck apakah operasi I/O nya udah selesai
   - klo blom, processor kasi signal ke I/O module buat siap-siap lagi, setelah kasi signal, processor kerjain yg lain
   - selagi processor kerjain yg lain, processor nunggu kedatangan interrupt dari I/O module lagi
## Footnote
### Spatial Locality
  - data yang di akses oleh processor itu biasanya berdekatan satu sama lain
  - jadi ketika sebuah word di akses, ada kemungkinan besar word lain yg berdekatan dengan word di akses itu juga bakal ikut kepake in the near future
### temporal locality
  - data yang di akses oleh processor itu biasanya di akses lagi di waktu yang dekat
  - jadi ketika sebuah word di akses, ada kemungkinan besar word yg sama bakal di akses lagi di waktu yang dekat
