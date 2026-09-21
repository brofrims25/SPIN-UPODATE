# Catatan dari Claude — status: BELUM di-build, BELUM dites di server beneran

Semua perubahan di sini berdasarkan pembacaan source code asli (GPL-3.0, milik
nulli0n/NightExpress), bukan hasil bongkar jar. Tapi sandbox yang aku pakai
tidak punya akses ke Maven Central / repo NightExpress / PaperMC, dan tidak
ada server Paper 26.1.2 beneran di sini — jadi belum ada `mvn compile` atau
uji jalan yang benar-benar terjadi. **Wajib build & tes sendiri** sebelum
dipakai di server produksi.

## 1. Perubahan yang sudah dilakukan (pom.xml)

| Dependency | Lama | Baru | Alasan |
|---|---|---|---|
| `nightcore` | 2.10.0 | 2.16.4 | Versi NightCore yang Hangar-nya resmi tercatat mendukung "1.21.8 - 26.2". Ini perubahan paling penting — inilah yang akan bikin compiler ketahuan kalau ada method/class NightCore lama yang sudah berubah. |
| `spigot-api` | 1.21.10-R0.1-SNAPSHOT | 1.21.11-R0.1-SNAPSHOT | Menyamakan dengan versi yang **NightCore 2.16.4 sendiri** masih pakai untuk compile (dicek langsung di pom.xml repo nightcore-spigot). Paper/Spigot belum punya artifact API bernomor "26.x" — 1.21.11 masih yang terakhir. |
| `packetevents-spigot` | 2.10.1 | 2.13.0 | Rilis resmi PacketEvents yang changelog-nya bilang "mainly support for Minecraft 26.2" (Juni 2026). |
| `ProtocolLib` | 5.3.0 | (tetap) | Nggak ketemu rilis lebih baru yang bisa dikonfirmasi. |

## 2. Cara build

Butuh **JDK 21+** dan **Maven** di komputer kamu (bukan cuma di server):

```bash
cd ExcellentCrates-spigot
mvn clean package
```

Jar hasil build ada di `target/ExcellentCrates-<versi>.jar`.

## 3. Kalau muncul error compile

Kemungkinan besar errornya `cannot find symbol`, nunjuk ke kelas/method di
`su.nightexpress.nightcore.*`. Itu bagus — artinya kita ketemu persis bagian
kode yang perlu disesuaikan ke NightCore 2.16.4. **Kirim stack trace error
compile-nya**, nanti bisa dibetulkan bagian kode yang manggil API lama itu.

## 4. Soal armor stand / "Crate Hologram" yang hilang di GUI

Ini **bukan** dari kode yang aku ubah — ini soal dependency di runtime,
sudah gitu dari sono-nya (bukan bug):

- File: `src/main/java/su/nightexpress/excellentcrates/hologram/HologramManager.java`
- Method `detectHandler()` cuma ngecek apakah plugin **PacketEvents** atau
  **ProtocolLib** terpasang & terdeteksi di server. Kalau nggak ada satupun,
  dia nge-print warning di console dan icon armor stand ("Crate Hologram")
  disembunyikan dari menu Crate Settings.

Checklist:
1. Buka log server pas startup, cari baris:
   `You have no packet library plugins installed for the Holograms feature to work.`
2. Kalau muncul → pasang PacketEvents 2.13.0+ (disarankan, update-nya lebih
   rutin) atau ProtocolLib yang sudah dukung 26.1.2/26.2.
3. Kalau salah satunya sudah terpasang tapi armor stand masih hilang →
   kemungkinan versinya masih lebih lama dari yang dibutuhkan 26.1.2 — update.

## 5. Info yang masih dibutuhkan buat lanjutin fix "banyak fitur rusak"

1. Log console server pas startup (baris soal ExcellentCrates, NightCore,
   PacketEvents/ProtocolLib)
2. Versi persis NightCore & PacketEvents/ProtocolLib yang terpasang sekarang
3. Fitur spesifik yang error + pesan error di console (kalau ada), misalnya
   pas buka GUI editor, pas crate dibuka pemain, dll.

## 6. Soal lisensi

ExcellentCrates & NightCore = GPL-3.0, oleh nulli0n / NightExpress:
- https://github.com/nulli0n/ExcellentCrates-spigot
- https://github.com/nulli0n/nightcore-spigot

Fork ini sah dipush ke GitHub kamu sendiri, asal file `LICENSE` tetap ada dan
kredit ke pembuat aslinya jelas. Ada juga 2 issue open di repo asli soal
masalah yang sama di versi 26.2 (#334, #332, belum direspons developernya) —
kalau fix ini beneran jalan, lumayan kalau di-share balik ke sana juga buat
bantu pengguna lain.
