# Tutorial Setup Integrasi ClickUp di ChatGPT (Akun ClickUp Free)

Panduan lengkap langkah demi langkah untuk menghubungkan **ClickUp** ke **ChatGPT**, dioptimalkan khusus untuk pengguna akun **ClickUp Free (Free Forever)**.

---

## 1. Perbandingan 2 Metode Integrasi ClickUp di ChatGPT

Karena **ChatGPT** berjalan di *cloud* (`https://chatgpt.com`), terdapat **2 metode resmi** untuk menghubungkan ChatGPT ke ClickUp dengan perbedaan kuota (*rate limit*) yang sangat penting bagi pengguna akun **ClickUp Free**:

| Metode Integrasi di ChatGPT | Jenis API & Autentikasi | Kuota pada Akun ClickUp Free | Kelebihan & Rekomendasi |
|---|---|---|---|
| **Metode 1: Custom GPT + Actions (OpenAPI Schema)** | ClickUp REST API v2 & v3 (`Personal API Token: pk_...`) | **100 request / menit** *(reset tiap menit, tanpa batas harian)* | **Sangat Direkomendasikan** untuk penggunaan intensif (baca/buat banyak Task & baca ClickUp Docs tanpa takut kuota harian habis). |
| **Metode 2: ChatGPT Custom MCP Connector** | ClickUp Official Remote MCP (`https://mcp.clickup.com/mcp` via OAuth 2.1) | **Hanya 100 call / 24 jam** *(kuota gabungan satu workspace)* | Setup paling cepat (tanpa input skema OpenAPI), namun kuota harian cepat habis jika dipakai intensif. |

> **Prasyarat Akun ChatGPT:** Kedua metode di atas membutuhkan akun **ChatGPT Plus, Pro, Team, atau Enterprise** untuk membuat Custom GPT atau menambahkan Custom MCP Connector.

---

## 2. Metode 1: Setup Menggunakan Custom GPT + Actions (Kuota 100 Request/Menit)

Metode ini menggunakan **Personal API Token (`pk_...`)** sehingga Anda mendapatkan kuota penuh **100 request per menit** meskipun menggunakan akun **ClickUp Free**.

### Tahap A: Membuat Personal API Token & Mengetahui ID di ClickUp

1. **Membuat Personal API Token (`pk_...`):**
   - Login ke akun **ClickUp** di browser (`https://app.clickup.com`).
   - Klik **Avatar / Profil** Anda (di pojok kiri bawah atau kanan atas) → pilih **Settings**.
   - Pada menu sidebar kiri, klik **ClickUp API**.
   - Di bagian **Personal API Token**, klik tombol **Create token** (atau **Generate**).
   - Salin (*copy*) token yang dihasilkan (selalu berawalan `pk_...`, contoh: `pk_12345678_XXXXXXXXXXXXXXXXXXXXXXXXXXXX`).

2. **Mengetahui `Team ID` (Workspace ID) dari URL Browser:**
   - Saat membuka halaman utama ClickUp di browser, perhatikan deretan angka pertama setelah domain `app.clickup.com/`:
     ```text
     https://app.clickup.com/9018645826/v/l/...
                             ^^^^^^^^^^
                       Ini adalah Team / Workspace ID Anda
     ```

---

### Tahap B: Membuat Custom GPT di ChatGPT

1. Buka **ChatGPT** (`https://chatgpt.com`).
2. Di sidebar kiri, klik **Explore GPTs** → klik tombol **+ Create** di pojok kanan atas.
3. Pilih tab **Configure** di bagian atas, lalu isi identitas asisten Anda:
   - **Name:** `ClickUp Workspace Assistant`
   - **Description:** `Asisten manajemen Task, Project, dan pembaca dokumentasi ClickUp Workspace.`
   - **Instructions:**
     ```text
     Kamu adalah asisten teknis yang terhubung langsung ke ClickUp Workspace melalui REST API v2 dan v3.
     Default Workspace/Team ID adalah 9018645826 dan Space utama (CAKRA) memiliki ID 90188195605.

     Panduan kerja:
     1. Gunakan action yang tersedia untuk mengecek daftar Space, Folder, List, dan Task.
     2. Untuk membaca dokumentasi di ClickUp Docs, panggil getFolderViews atau listWorkspaceDocs untuk mendapatkan doc_id, lalu panggil getDocPageListing untuk melihat daftar halaman (page_id), dan getDocPageContent untuk membaca isi lengkap halamannya.
     3. Karena menggunakan akun ClickUp Free, hindari penggunaan Custom Fields saat membuat task baru. Gunakan parameter markdown_description, priority (1=Urgent, 2=High, 3=Normal, 4=Low), dan tags.
     ```

---

### Tahap C: Mengatur Authentication (Penting!)

1. Masih di tab **Configure**, scroll ke paling bawah ke bagian **Actions** → klik tombol **Create new action**.
2. Pada bagian **Authentication**, klik ikon pengaturan (⚙️):
   - **Authentication Type:** pilih **API Key**
   - **Auth Type:** pilih **Custom**
     > ⚠️ **Penting:** Jangan memilih *Bearer*! API ClickUp mewajibkan header `Authorization: pk_...` dikirim langsung tanpa kata `Bearer` di depannya.
   - **Custom Header Name:** ketik `Authorization`
   - **API Key:** tempelkan (*paste*) Personal API Token Anda (`pk_...`).
3. Klik **Save**.

---

### Tahap D: Memasukkan OpenAPI Schema

Salin dan tempelkan spesifikasi **OpenAPI 3.1.0** berikut ke dalam kotak **Schema** di bawah pengaturan Authentication:

```yaml
openapi: 3.1.0
info:
  title: ClickUp API v2 and v3 Integration
  description: REST API schema for managing ClickUp Spaces, Folders, Lists, Tasks, and reading ClickUp Docs.
  version: 1.0.0
servers:
  - url: https://api.clickup.com
paths:
  /api/v2/team/{team_id}/space:
    get:
      operationId: getSpaces
      summary: List all spaces in the workspace
      parameters:
        - name: team_id
          in: path
          required: true
          schema:
            type: string
            default: "9018645826"
      responses:
        "200":
          description: Successful response
  /api/v2/space/{space_id}/folder:
    get:
      operationId: getFolders
      summary: List all folders in a space
      parameters:
        - name: space_id
          in: path
          required: true
          schema:
            type: string
            default: "90188195605"
      responses:
        "200":
          description: Successful response
  /api/v2/folder/{folder_id}:
    get:
      operationId: getFolderDetails
      summary: Get folder details and lists inside a folder
      parameters:
        - name: folder_id
          in: path
          required: true
          schema:
            type: string
      responses:
        "200":
          description: Successful response
  /api/v2/folder/{folder_id}/view:
    get:
      operationId: getFolderViews
      summary: Get views and ClickUp Docs attached to a folder
      parameters:
        - name: folder_id
          in: path
          required: true
          schema:
            type: string
      responses:
        "200":
          description: Successful response
  /api/v2/list/{list_id}/task:
    get:
      operationId: getTasks
      summary: Get tasks inside a list
      parameters:
        - name: list_id
          in: path
          required: true
          schema:
            type: string
        - name: include_closed
          in: query
          required: false
          schema:
            type: boolean
            default: true
      responses:
        "200":
          description: Successful response
    post:
      operationId: createTask
      summary: Create a new task inside a list
      parameters:
        - name: list_id
          in: path
          required: true
          schema:
            type: string
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - name
              properties:
                name:
                  type: string
                  description: Task title
                markdown_description:
                  type: string
                  description: Task description in Markdown format
                priority:
                  type: integer
                  description: "1 = Urgent, 2 = High, 3 = Normal, 4 = Low"
                tags:
                  type: array
                  items:
                    type: string
      responses:
        "200":
          description: Task created
  /api/v3/workspaces/{workspace_id}/docs:
    get:
      operationId: listWorkspaceDocs
      summary: List all ClickUp Docs in the workspace
      parameters:
        - name: workspace_id
          in: path
          required: true
          schema:
            type: string
            default: "9018645826"
      responses:
        "200":
          description: Successful response
  /api/v3/workspaces/{workspace_id}/docs/{doc_id}/pageListing:
    get:
      operationId: getDocPageListing
      summary: Get the hierarchy of pages and subpages inside a ClickUp Doc
      parameters:
        - name: workspace_id
          in: path
          required: true
          schema:
            type: string
            default: "9018645826"
        - name: doc_id
          in: path
          required: true
          schema:
            type: string
      responses:
        "200":
          description: Successful response
  /api/v3/workspaces/{workspace_id}/docs/{doc_id}/pages/{page_id}:
    get:
      operationId: getDocPageContent
      summary: Read the full content of a specific page inside a ClickUp Doc
      parameters:
        - name: workspace_id
          in: path
          required: true
          schema:
            type: string
            default: "9018645826"
        - name: doc_id
          in: path
          required: true
          schema:
            type: string
        - name: page_id
          in: path
          required: true
          schema:
            type: string
      responses:
        "200":
          description: Successful response
```

---

### Tahap E: Simpan & Uji Custom GPT Anda

1. Klik tombol **Create** (atau **Update**) di pojok kanan atas.
2. Pilih akses **Only me** (atau **Anyone with a link** jika ingin dibagikan ke rekan setim).
3. Klik **Save**, lalu uji di ruang chat Custom GPT dengan prompt seperti:
   - *"Cek folder apa saja yang ada di Space CAKRA"*
   - *"Di folder Documentation, ada dokumen apa saja beserta daftar halamannya?"*
   - *"Tolong bacakan isi halaman Executive Summary pada dokumen STREAMING PLATFORM V1"*

---

## 3. Metode 2: Setup Menggunakan ChatGPT Custom MCP Connector (OAuth Resmi)

Jika Anda ingin menghubungkan ClickUp langsung ke chat reguler ChatGPT menggunakan protokol MCP resmi dari ClickUp (`https://mcp.clickup.com/mcp`):

### Langkah Setup:
1. Buka **ChatGPT** (`https://chatgpt.com`) → klik **Profil Anda** → pilih **Settings**.
2. Masuk ke menu **Connectors** (atau **Connected Apps**).
3. Scroll ke bawah ke bagian **Advanced**, lalu aktifkan toggle **Developer Mode** (agar opsi penambahan Custom MCP Server muncul).
4. Klik tombol **Create** / **Add Custom MCP Server**, kemudian isi form berikut:
   - **Name:** `ClickUp`
   - **Server URL:** `https://mcp.clickup.com/mcp`
   - **Authentication:** pilih **OAuth**
5. Klik **Connect / Authenticate**.
6. Anda akan diarahkan ke halaman login ClickUp di browser → pilih Workspace Anda → klik **Allow Access**.
7. Saat membuka ruang chat baru di ChatGPT, klik ikon **Tools (`+`)** di kolom chat dan pastikan tool **ClickUp** diaktifkan.

> **Catatan Kuota Metode 2 pada Akun ClickUp Free:**
> Server MCP resmi ClickUp (`mcp.clickup.com`) menerapkan batas **100 call per 24 jam** untuk seluruh workspace pada paket Free Forever. Jika kuota harian habis (`429 RATE_LIMIT_EXCEEDED`), gunakan **Metode 1 (Custom GPT + Actions)** yang memiliki kuota 100 request per menit.

---

## 4. Tips & Praktik Terbaik untuk Akun ClickUp Free di ChatGPT

Agar instruksi dari ChatGPT ke ClickUp berjalan lancar tanpa terbentur batasan fitur paket **ClickUp Free Forever**:

1. **Hindari Penggunaan *Custom Fields*:**
   - Paket ClickUp Free membatasi penggunaan *Custom Fields* maksimal **60 kali** di seluruh workspace.
   - **Solusi:** Mintalah ChatGPT menyimpan metadata teknis (seperti ID Requirement `REQ-2026-001`, komponen `K-01`, nama microservice) ke dalam **Task Tags** (*unlimited* di paket Free), **Native Priority** (`1: Urgent`, `2: High`, `3: Normal`, `4: Low`), serta tabel Markdown di dalam **Task Description**.
2. **Efisiensi Struktur Space & List:**
   - Paket Free memiliki batas maksimal **5 Spaces** per Workspace dan **100 Lists** per Space. Fokuskan pembuatan proyek di dalam **Folder** dan **List** pada satu Space utama.
3. **Hemat Kuota Penyimpanan (Batas 100 MB):**
   - Jangan mengunggah lampiran dokumen besar ke ClickUp. Cukup tautkan URL dokumen atau link repositori Git di dalam deskripsi Task atau halaman ClickUp Doc.
