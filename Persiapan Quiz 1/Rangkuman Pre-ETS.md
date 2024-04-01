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
Flow:
1. Process beroperasi
2. Interruption muncul
3. Pertama dihandle oleh `Interruption Handler X`
4. IHX (Interruption Handler X) return/selesai handle interrupt
5. Selesai IHX, dilanjutkan oleh `Interruption Handler Y` (mereka bekerja pas after one another, gaada break)
6. IHY return/selesai handle interrupt
7. Selesai IHY, process berjalan lagi
### Nested Interrupt Programming
1. Process beroperasi
2. Interruption muncul
3. Pertama dihandle oleh `Interruption Handler X`
4. Di dalam IHX ada interrupt lagi yang di handle oleh `Interruption Handler Y`
5. IHY selesai, kembali ke IHX
6. IHX selesai, kembali ke process
7. process berjalan lagi
## Multiple Interrupt
