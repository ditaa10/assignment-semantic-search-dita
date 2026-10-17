# Laporan Pengujian Retrieval dan Respon LLM (RAG System)

Dokumen ini berisi catatan pengujian 5 query pada sistem RAG (Retrieval-Augmented Generation) berbasis FAISS dan Gemini AI.

---

## 1. Ringkasan Hasil Pengujian

| No | Tipe Query | Input Query | FAQ Teratas (FAISS Retrieval) | Respon LLM | Status / Analisis |
|---|---|---|---|---|---|
| **1** | **Exact Match** | *"Bagaimana cara melakukan pemesanan di platform ini?"* | *"Bagaimana cara melakukan pemesanan di platform ini?"* | Memberikan langkah-langkah pemesanan yang akurat sesuai FAQ. | **Sukses:** Retrieval 100% presisi, respon LLM sesuai context. |
| **2** | **Exact Match** | *"Apakah saya bisa memesan tanpa membuat akun?"* | *"Apakah saya bisa memesan tanpa membuat akun?"* | Menjelaskan bahwa akun diperlukan untuk melacak status pesanan. | **Sukses:** Retrieval tepat sasaran. |
| **3** | **Parafrase / Sinonim** | *"gimana cara order di platform ini?"* | *"Bagaimana cara melakukan pemesanan di platform ini?"* | Memberikan petunjuk alur order/pemesanan dengan tepat. | **Sukses:** Semantic search mengenali kata *"order"* $\rightarrow$ *"pemesanan"*. |
| **4** | **Parafrase / Sinonim** | *"bisa pesan tanpa punya akun?"* | *"Apakah saya bisa memesan tanpa membuat akun?"* | Mengonfirmasi bahwa akun wajib dibuat terlebih dahulu. | **Sukses:** Menangkap makna kalimat tanya yang disingkat. |
| **5** | **Out-of-Domain** | *"Apakah toko ini menjual hewan peliharaan kucing persia?"* | *"Apakah saya bisa request custom produk kepada penjual?"* | Menyatakan maaf karena informasi tidak tersedia & menyarankan hubungi CS. | **Sukses (Guardrail):** LLM tidak berhalusinasi meskipun FAISS melakukan *forced retrieval*. |

---

## 2. Analisis & Pengamatan Utama

1. **Efektivitas Semantic Search (Query 1–4):**
   * FAISS berhasil menangkap makna secara kontekstual. Query berbasis bahasa informal/sinonim seperti *"gimana cara order"* tetap menarik FAQ resmi *"Bagaimana cara melakukan pemesanan..."* karena jarak vektor embeddingnya yang dekat.

2. **Perilaku Out-of-Domain & Anti-Halusinasi (Query 5):**
   * **Masalah FAISS:** Karena mengandalkan kalkulasi jarak L2 (`IndexFlatL2`), FAISS secara matematis **memaksa** mengambil $k=3$ dokumen terdekat (dalam kasus ini FAQ *custom produk*), meskipun topiknya tidak relevan.
   * **Mitigasi Prompt:** Instruksi ketat pada `build_prompt` (*"Jawab HANYA berdasarkan informasi FAQ..."*) berhasil mencegah LLM memanfaatkan context yang tidak relevan tersebut, sehingga LLM secara jujur menjawab bahwa informasi tidak ditemukan.