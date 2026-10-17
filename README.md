# Assignment: Semantic Search & RAG Assistant for E-Commerce FAQ

Proyek ini mengimplementasikan sistem **Retrieval-Augmented Generation (RAG)** sederhana untuk menjawab pertanyaan pelanggan e-commerce secara otomatis berdasarkan *Knowledge Base* FAQ.

---

## 1. Problem Statement & Knowledge Base
* **Problem:** Asisten AI konvensional sering mengalami halusinasi dan tidak mengetahui konteks spesifik toko e-commerce.
* **Knowledge Base:** Berisi 59 entri FAQ e-commerce (pertanyaan, jawaban, dan kategori) yang digabungkan menjadi unit retrievable `Q: {question} A: {answer}`.

---

## 2. Alur Sistem
1. **Ingest:** Membaca data CSV menggunakan `pandas` dan merakit unit teks retrievable.
2. **Embed:** Mengubah teks menjadi vektor embedding dimensi 3072 menggunakan `gemini-embedding-001` (`float32`).
3. **Store:** Menyimpan matriks embedding ke dalam vector index `faiss.IndexFlatL2`.
4. **Retrieve:** Mencari $k=3$ dokumen FAQ paling relevan berdasarkan kalkulasi jarak L2 saat query diterima.
5. **Generate:** Menggabungkan query dan hasil retrieval ke dalam prompt terstruktur, lalu mengirimkannya ke `gemini-2.0-flash` untuk merangkai jawaban akhir.

---

## 3. Hasil Pengujian & Analisis Relevansi

* **Query Exact Match:** *"Bagaimana cara melakukan pemesanan di platform ini?"*
  * **Hasil:** Retrieval 100% tepat pada sumber utama. LLM merespons dengan akurat.
* **Query Parafrase / Sinonim:** *"gimana cara order di platform ini?"* & *"bisa pesan tanpa punya akun?"*
  * **Hasil:** Semantic search berhasil memahami makna kata "order" -> "pemesanan" dan menarik dokumen yang relevan meskipun kosa katanya berbeda.
* **Query Out-of-Domain:** *"Apakah toko ini menjual hewan peliharaan kucing persia?"*
  * **Hasil:** *Guardrails* berjalan. LLM secara jujur menyatakan informasi tidak tersedia dan mengarahkan pengguna ke Customer Service.

---

## 4. Keterbatasan Sistem
- FAISS Selalu Memaksa Mengembalikan Dokumen (Forced Retrieval):
Kondisi: Saat diuji query di luar knowledge base (misal: "kucing persia"), FAISS tetap mencocokkan dan menarik dokumen teratas (misal: FAQ "custom produk"), karena FAISS secara matematis wajib mengembalikan indeks terdekat tanpa tahu apakah data itu benar-benar ada atau tidak.

- Bergantung Penuh pada Prompt Guardrails LLM:
Kondisi: Karena FAISS tidak bisa bilang "data kosong", sistem kita sepenuhnya bergantung pada instruksi teks di build_prompt agar LLM menolak menjawab dan mencegah terjadinya halusinasi AI.

---

## 5. Cara Menjalankan Notebook/Script

1. **Install Dependensi:**
   ```bash
   pip install pandas numpy faiss-cpu google-genai
   ```
2. **Set API Key:**
   ```bash
   export GEMINI_API_KEY="your_gemini_api_key_here"
   ```
3. **Jalankan Notebook:**
   Buka dan jalankan seluruh sel pada file notebook (misal: `assignment_rag.ipynb`) secara berurutan.