# Tutorial Setup Integrasi ClickUp di Google Antigravity & ChatGPT (Akun ClickUp Free)

Panduan lengkap langkah demi langkah untuk menghubungkan **ClickUp** ke **Google Antigravity** dan **ChatGPT**, dioptimalkan khusus untuk pengguna akun **ClickUp Free (Free Forever)**.

---

## 1. Perbandingan Metode Koneksi & Kuota pada Akun ClickUp Free

ClickUp membedakan kuota (*rate limit*) antara penggunaan **REST API (Personal API Token)** dan **Official Remote MCP Server**:

| Metode Integrasi | Jenis API | Platform AI yang Didukung | Kuota pada Akun ClickUp Free | Rekomendasi |
|---|---|---|---|---|
| **Local Stdio MCP (`@taazkareem/clickup-mcp-server`)** | ClickUp REST API v2 (`pk_...`) | **Google Antigravity**, Claude Desktop, Cursor | **100 request / menit** *(tanpa batas harian)* | **Terbaik untuk Antigravity** |
| **Custom GPT + Actions (OpenAPI Schema)** | ClickUp REST API v2 & v3 (`pk_...`) | **ChatGPT** (Plus / Team / Enterprise) | **100 request / menit** *(tanpa batas harian)* | **Terbaik untuk ChatGPT** (kuota lega) |
| **Official Remote MCP (`https://mcp.clickup.com/mcp`)** | ClickUp Hosted AI MCP (OAuth 2.1) | **ChatGPT Connectors**, Antigravity, Claude Web | **Hanya 100 call / 24 jam** *(satu workspace)* | Praktis untuk setup cepat, namun kuota harian terbatas |

---

## 2. Persiapan Awal: Membuat Personal API Token & Team ID di ClickUp

Langkah ini digunakan baik untuk **Google Antigravity** maupun **ChatGPT (Custom GPT Actions)**.

### 2.1 Cara Membuat Personal API Token (`CLICKUP_API_KEY`)
1. Login ke akun **ClickUp** Anda melalui browser (`https://app.clickup.com`).
2. Klik **Avatar / Profil** Anda (di pojok kiri bawah atau kanan atas) → pilih **Settings**.
3. Pada menu sidebar kiri, masuk ke menu **ClickUp API**.
4. Di bagian **Personal API Token**, klik tombol **Create token** (atau **Generate**).
5. Salin (*copy*) token yang muncul (selalu diawali dengan `pk_...`, contoh: `pk_12345678_XXXXXXXXXXXXXXXXXXXXXXXXXXXX`).

> **Catatan Keamanan:** Jangan pernah membagikan atau melakukan *commit* token `pk_...` asli ke dalam repositori Git. Simpan hanya di konfigurasi lokal atau pengaturan rahasia (*Authentication*) pada klien AI Anda.

### 2.2 Cara Mendapatkan `CLICKUP_TEAM_ID` (Workspace ID)
- **Cara A (Dari URL Browser):** Buka halaman utama ClickUp di browser, lalu lihat deretan angka pertama setelah domain `app.clickup.com/`:
  ```text
  https://app.clickup.com/9018645826/v/l/...
                          ^^^^^^^^^^
                    Ini adalah CLICKUP_TEAM_ID Anda
  ```
- **Cara B (Otomatis via Terminal PowerShell):**
  ```powershell
  python -c "import urllib.request, json; req = urllib.request.Request('https://api.clickup.com/api/v2/team', headers={'Authorization': 'pk_ISI_TOKEN_ANDA'}); print(json.loads(urllib.request.urlopen(req).read().decode()))"
  ```

---

## 3. Bagian I: Setup MCP ClickUp di Google Antigravity (Windows)

Karena **Antigravity** berjalan secara lokal di komputer Anda, Antigravity dapat menjalankan server MCP berbasis Node.js (`npx`) secara langsung.

### Langkah 1: Pastikan Node.js & `npx` Terinstal
Cek melalui PowerShell:
```powershell
node -v
npx -v
```

### Langkah 2: Edit File `mcp_config.json`
Perbarui file konfigurasi global Antigravity di:
- `C:\Users\<username>\.gemini\config\mcp_config.json`
- `C:\Users\<username>\.gemini\antigravity\mcp_config.json`

Isi dengan konfigurasi berikut (menggunakan wrapper `cmd /c npx` agar proses Stdio berjalan stabil di Windows):

```json
{
  "mcpServers": {
    "clickup": {
      "command": "cmd",
      "args": [
        "/c",
        "npx",
        "-y",
        "@taazkareem/clickup-mcp-server@latest"
      ],
      "env": {
        "CLICKUP_API_KEY": "pk_ISI_DENGAN_PERSONAL_API_TOKEN_ANDA",
        "CLICKUP_TEAM_ID": "ISI_DENGAN_TEAM_ID_ANDA"
      }
    }
  }
}
```

### Langkah 3: Aktivasi di UI Antigravity
1. Buka panel chat **Antigravity**.
2. Klik ikon **Additional Options (`...`)** di pojok kanan atas panel chat → pilih **MCP Servers**.
3. Klik tombol **Refresh / Reconnect** pada server **`clickup`** hingga statusnya aktif.

---

## 4. Bagian II: Setup ClickUp di ChatGPT

Karena **ChatGPT** berjalan di *cloud* (`https://chatgpt.com`), ChatGPT tidak dapat memanggil proses `npx` lokal di laptop Anda. Berikut **2 cara** menghubungkan ClickUp ke ChatGPT:

---

### Cara 1: Menggunakan Fitur Custom MCP Connector di ChatGPT (OAuth Resmi)

Cocok untuk setup instan tanpa konfigurasi skema API, tersedia pada paket ChatGPT Plus / Pro / Team / Enterprise.

#### Langkah Setup:
1. Buka **ChatGPT** (`https://chatgpt.com`) → klik **Profil Anda** → **Settings**.
2. Masuk ke menu **Connectors** (atau **Connected Apps**).
3. Scroll ke bagian **Advanced**, lalu aktifkan toggle **Developer Mode** (diperlukan agar tombol tambah Custom MCP muncul).
4. Klik tombol **Create** / **Add Custom MCP Server**, lalu isi parameter berikut:
   - **Name:** `ClickUp`
   - **Server URL:** `https://mcp.clickup.com/mcp`
   - **Authentication:** Pilih **OAuth**
5. Klik **Connect / Authenticate** → halaman otorisasi ClickUp akan terbuka di browser → pilih Workspace Anda → klik **Allow**.
6. Saat memulai chat di ChatGPT, klik ikon **Tools (`+`)** di kolom input chat dan pastikan koneksi **ClickUp** diaktifkan.

> **Batasan Cara 1 pada Akun ClickUp Free:** Koneksi ke `https://mcp.clickup.com/mcp` dibatasi **100 call per 24 jam** untuk seluruh workspace.

---

### Cara 2: Menggunakan Custom GPT + Actions (Pakai Token `pk_...` — Kuota 100 req/menit)

Jika Anda ingin menggunakan **Personal API Token (`pk_...`)** di ChatGPT agar mendapatkan kuota **100 request per menit** (bebas dari batas harian 100 call/24 jam) sekaligus bisa membaca isi halaman **ClickUp Docs**, buatlah **Custom GPT**:

#### Langkah 1: Buat Custom GPT Baru
1. Buka **ChatGPT** (`https://chatgpt.com`) → klik **Explore GPTs** di sidebar kiri.
2. Klik tombol **+ Create** di pojok kanan atas.
3. Pilih tab **Configure**:
   - **Name:** `ClickUp Workspace Assistant`
   - **Description:** `Asisten manajemen Task dan pembaca dokumentasi ClickUp Workspace.`
   - **Instructions:**
     ```text
     Kamu adalah asisten teknis yang terhubung ke ClickUp Workspace melalui REST API v2 dan v3.
     Default Team/Workspace ID adalah 9018645826 dan Space CAKRA adalah 90188195605.
     Gunakan actions yang tersedia untuk membaca daftar Space, Folder, List, Task, serta membaca daftar dan isi halaman ClickUp Docs.
     ```

#### Langkah 2: Konfigurasi Authentication (Penting!)
1. Di tab **Configure**, scroll ke paling bawah ke bagian **Actions** → klik **Create new action**.
2. Klik kolom **Authentication** (ikon ⚙️):
   - **Authentication Type:** pilih **API Key**
   - **Auth Type:** pilih **Custom** *(JANGAN pilih Bearer, karena ClickUp API mewajibkan header `Authorization: pk_...` tanpa awalan kata `Bearer`)*
   - **Custom Header Name:** ketik `Authorization`
   - **API Key:** masukkan Personal API Token Anda (`pk_...` hasil dari menu **Settings → ClickUp API → Create token**)
3. Klik **Save**.

#### Langkah 3: Masukkan OpenAPI Schema
Salin dan tempelkan spesifikasi **OpenAPI 3.1.0** berikut ke dalam kotak **Schema**:

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

#### Langkah 4: Simpan & Publikasikan Secara Privat
1. Klik tombol **Create** (atau **Update**) di pojok kanan atas.
2. Pilih visibilitas **Only me** (atau **Anyone with a link** di dalam tim internal).
3. Klik **Save** dan uji Custom GPT Anda!

---

## 5. Tips & Praktik Terbaik untuk Akun ClickUp Free Forever

Agar penggunaan integrasi AI di **Antigravity** maupun **ChatGPT** berjalan mulus tanpa terkena pembatasan fitur akun Free:

1. **Hindari Ketergantungan pada *Custom Fields*:**
   - Paket ClickUp Free membatasi penggunaan *Custom Fields* maksimal **60 kali** di seluruh workspace.
   - **Solusi:** Simpan atribut teknis (seperti ID Requirement `REQ-2026-001`, komponen `K-01`, nama microservice) menggunakan **Task Tags** (*unlimited* di paket Free), **Native Priority** (`1: Urgent`, `2: High`, `3: Normal`, `4: Low`), serta tabel Markdown di dalam **Task Description**.
2. **Efisiensi Struktur Space & List:**
   - Paket Free memiliki batas maksimal **5 Spaces** per Workspace dan **100 Lists** per Space. Gunakan pengelompokan **Folder** dan **List** di dalam satu Space utama (misal: Space `CAKRA`).
3. **Hemat Kuota Penyimpanan (Batas 100 MB):**
   - Hindari mengunggah lampiran file berukuran besar ke ClickUp. Cukup tautkan URL dokumen atau path repositori Git (`https://gitops.tritronik.com/...`) di dalam deskripsi Task atau ClickUp Doc.

