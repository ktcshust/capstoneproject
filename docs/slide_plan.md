# Kế hoạch Slide — Đồ án Capstone
# Knowledge Graph Integration for Heterogeneous Medical Data

> **Thời lượng đề xuất:** 15 phút thuyết trình + 5 phút Q&A  
> **Số slide:** ~22 slide  
> **Công cụ gợi ý:** PowerPoint / Google Slides / Canva  
> **Màu chủ đạo:** Xanh navy (`#2C3E50`) + Xanh nhạt (`#3498DB`) + Trắng — chuyên nghiệp, kỹ thuật

---

## SLIDE 1 — COVER (0:00–0:30)

**Tiêu đề lớn:** Knowledge Graph Integration  
**Phụ đề:** for Heterogeneous Medical Data

**Thông tin:**
- Tên nhóm / thành viên
- Tên môn học / giảng viên hướng dẫn
- Ngày bảo vệ

**Hình ảnh:**
- Background: ảnh mờ của mạng lưới đồ thị (có thể dùng screenshot của pyvis viz)
- Hoặc logo Neo4j + icon y khoa (tim, thuốc, bệnh viện)

**Design tip:** Chữ trắng nổi bật trên background tối/mờ.

---

## SLIDE 2 — VẤN ĐỀ & ĐỘNG LỰC (0:30–1:30)

**Tiêu đề:** Tại sao cần tích hợp dữ liệu y khoa?

**Nội dung (3 bullet points):**
- 📋 Bác sĩ có triệu chứng → phải tra nhiều hệ thống khác nhau để ra quyết định
- 🗂️ Dữ liệu y khoa tồn tại dạng **rời rạc**: bảng biểu thuốc, danh sách triệu chứng, ghi chú lâm sàng
- 🔍 Không nguồn nào đơn lẻ có đủ thông tin để phát hiện **mối quan hệ ẩn**

**Hình ảnh (quan trọng):**
```
[Hình: 3 "hộp" dữ liệu riêng biệt với dấu ? ở giữa]
  ┌─────────────┐    ┌─────────────┐    ┌──────────────┐
  │  Drug CSV   │    │Disease-Symp │    │ Ghi chú lâm  │
  │ 1,420 thuốc │  ? │  41 bệnh    │  ? │  sàng 5,000  │
  │ 5,400 SE    │    │ 131 triệu   │    │  transcripts │
  └─────────────┘    └─────────────┘    └──────────────┘
            Không có liên kết → thông tin bị cô lập
```
→ Vẽ đơn giản trong PowerPoint với shape + icon

**Speaker note:** Mở đầu bằng câu hỏi: *"Bệnh nhân vào viện với chest pain — bác sĩ cần biết gì trong 60 giây?"*

---

## SLIDE 3 — HOOK: USE CASE (1:30–2:30)

**Tiêu đề:** Bài toán thực tế: Bệnh nhân có "chest pain"

**Layout: TRƯỚC vs SAU (2 cột)**

| TRƯỚC (không có KG) | SAU (có KG) |
|---------------------|-------------|
| Tra 3 hệ thống khác nhau | 1 câu Cypher query |
| Kết quả rời rạc, không liên kết | Đồ thị: Symptom → Disease → Drug |
| Không thấy drug repurposing | Phát hiện aspirin có thể treat hypertension |

**Hình ảnh:**
- Screenshot kết quả query từ Neo4j Browser (query chest pain)
- Hoặc bảng kết quả nhỏ: Disease=hypertension, Drug=betaxolol, Rating=10.0

**Speaker note:** *"Đây là điều KG của chúng tôi làm được — trong < 100ms."*

---

## SLIDE 4 — GIẢI PHÁP TỔNG QUAN (2:30–3:00)

**Tiêu đề:** Knowledge Graph: Tích hợp tất cả vào một đồ thị

**1 hình trung tâm (quan trọng nhất của bài):**

```
         [Drug] ──TREATS──► [Disease] ──HAS_SYMPTOM──► [Symptom]
           │                    │
        CAUSES              TREATED_BY
           ▼                    ▼
       [SideEffect]        [Specialty]
           │
        BELONGS_TO
           ▼
        [DrugClass]

    Mọi node ←──── CO_MENTIONED (from NER) ────► Mọi node
```

**3 con số nổi bật:**
- **18,387** nodes
- **117,836** relationships  
- **3** nguồn dữ liệu tích hợp

**Design tip:** Dùng icon hoặc màu khác nhau cho mỗi node type. Đây là slide "wow" — làm nổi bật.

---

## SLIDE 5 — BA BỘ DỮ LIỆU (3:00–4:00)

**Tiêu đề:** Ba nguồn dữ liệu: 2 Structured + 1 Unstructured

**Layout: 3 card ngang (icon + tên + số liệu + vai trò)**

```
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│   💊 BỘ 1        │  │   🏥 BỘ 2        │  │   📝 BỘ 3        │
│ Drugs & Side     │  │ Disease-Symptom  │  │ Medical          │
│ Effects          │  │ Dataset          │  │ Transcriptions   │
│──────────────────│  │──────────────────│  │──────────────────│
│ 2,931 rows       │  │ 4 CSV files      │  │ 4,943 records    │
│ 1,420 Drug       │  │ 41 bệnh          │  │ 40 specialties   │
│ 5,447 SideEffect │  │ 131 Symptom      │  │ Free text NLP    │
│──────────────────│  │──────────────────│  │──────────────────│
│ Structured ✅    │  │ Structured ✅    │  │ Unstructured ⚡  │
└──────────────────┘  └──────────────────┘  └──────────────────┘
```

**Highlight quan trọng:** "Không bộ nào đơn độc đủ — mỗi bộ mang giá trị riêng biệt"

**Hình ảnh bổ sung (chọn 1):**
- Screenshot Excel/CSV mẫu của Bộ 1 (vài dòng đầu)
- Hoặc word cloud từ medical transcriptions

---

## SLIDE 6 — PIPELINE TỔNG THỂ (4:00–4:30)

**Tiêu đề:** Pipeline 8 Tuần: Từ Raw Data → Knowledge Graph

**1 hình flow ngang (quan trọng):**

```
  Bộ 1+2 CSV          Bộ 3 Text         ER              KG Build
┌──────────┐       ┌──────────┐    ┌──────────┐    ┌──────────────┐
│ Ingest   │──────►│   NER    │───►│ Entity   │───►│  Neo4j KG    │
│ Structured│      │scispaCy  │    │Resolution│    │ 18,387 nodes │
│ (Tuần 2) │       │(Tuần 3)  │    │(Tuần 4-5)│    │(Tuần 6)      │
└──────────┘       └──────────┘    └──────────┘    └──────┬───────┘
                                                           │
                                              ┌────────────┴──────────┐
                                              │                       │
                                        ┌─────▼──────┐        ┌──────▼─────┐
                                        │  Analytics │        │   Pyvis    │
                                        │    GDS     │        │    HTML    │
                                        │ (Tuần 7)   │        │ (Tuần 8)   │
                                        └────────────┘        └────────────┘
```

**Ghi chú dưới slide:** Tuần 1: EDA | Tuần 2: Ingest | Tuần 3: NER | Tuần 4: Co-occur | Tuần 5: ER | Tuần 6: Build KG | Tuần 7: Analytics | Tuần 8: Viz

**Speaker note:** Đây là overview — các slide sau sẽ đi sâu từng bước.

---

## SLIDE 7 — ONTOLOGY DESIGN (4:30–5:00)

**Tiêu đề:** Thiết kế Ontology: 7 Node Types × 7 Relationship Types

**2 phần:**

**Bên trái — Sơ đồ đồ thị (vẽ lại đẹp):**
```
DrugClass ◄── BELONGS_TO ──── Drug ──── CAUSES ──► SideEffect
                               │
                             TREATS
                               │
                               ▼
                           Disease ──── TREATED_BY ──► Specialty
                               │
                           HAS_SYMPTOM
                               │
                               ▼
                           Symptom

              CO_MENTIONED (bất kỳ cặp node nào — từ NER)
```

**Bên phải — Bảng thống kê nhỏ:**

| Node | Count | Source |
|------|-------|--------|
| Disease | 5,621 | Bộ 1+2+3 |
| Drug | 3,032 | Bộ 1+3 |
| SideEffect | 5,447 | Bộ 1 |
| Symptom | 131 | Bộ 2+3 |
| Specialty | 40 | Bộ 3 |

**Highlight:** Property `source[]` trên mọi node → truy vết nguồn gốc dữ liệu.

---

## SLIDE 8 — NER PIPELINE (5:00–5:45)

**Tiêu đề:** NLP Pipeline: Từ ghi chú lâm sàng → Entity

**Flow diagram + ví dụ thực tế:**

```
Input text:
"Patient presents with HTN and DM. Started on metformin..."

         ↓ scispaCy en_ner_bc5cdr_md
         
Entities raw: [HTN, DM, metformin]

         ↓ ABBREVIATION_MAP (dictionary)
         
Canonical: [hypertension, diabetes mellitus, metformin]

         ↓ GENERIC_BLACKLIST filter
         
Output: mentions.parquet (88,627 mentions, 4,943 docs)
```

**Bảng kết quả NER:**

| Entity Type | Mentions | Unique canonical |
|-------------|----------|-----------------|
| Drug (CHEMICAL) | ~42,000 | ~3,000 |
| Disease | ~46,000 | ~5,600 |

**Hình ảnh gợi ý:**
- Screenshot scispaCy displacy visualization (highlight entities trong text)
- Có thể chạy: `python -c "import spacy; nlp=spacy.load('en_ner_bc5cdr_md'); doc=nlp('HTN and metformin'); from spacy import displacy; displacy.render(doc, style='ent')"` rồi chụp màn hình

---

## SLIDE 9 — ENTITY RESOLUTION (5:45–6:45)

**Tiêu đề:** Entity Resolution: Gộp các biến thể tên về một thực thể

**Vấn đề (hình minh họa):**
```
"htn"  "HTN"  "high blood pressure"  "hypertensive disease"
    ↘      ↓           ↓                    ↙
              "hypertension" (1 node)
```

**Pipeline 2 tầng:**

```
Input: 3,600 unique entity mentions
       │
       ▼
[Tầng 1: Blocking]  →  Block key = 3 ký tự + Metaphone
       │               Giảm O(n²) → O(n·k)
       ▼
[Tầng 2: Matching]  →  Jaro-Winkler (lexical) × 0.4
       │               + Cosine Embedding (semantic) × 0.6
       ▼
[Auto-merge ≥ 0.85] +  [Dictionary: HTN→hypertension, Tylenol→acetaminophen]
       │
       ▼
Output: 8,017 mappings (684 real merges + 7,321 cosmetic)
```

**Highlight:** 2 loại dictionary bổ sung:
- Abbreviation Map: 22 entries (HTN, DM, CAD, COPD...)
- Brand-to-Generic: 15 entries (Tylenol→acetaminophen, Coumadin→warfarin...)

---

## SLIDE 10 — ER EVALUATION (6:45–7:15)

**Tiêu đề:** Đánh giá Entity Resolution trên Gold Set

**2 phần:**

**Bên trái — Phương pháp:**
- Gold Set: 150 mention annotated thủ công (stratified sampling)
- Evaluate: Compare similarity score vs. manual annotation

**Bên phải — Kết quả (bảng nổi bật):**

| Metric | Kết quả | Mục tiêu | |
|--------|---------|-----------|--|
| Precision | **1.00** | ≥ 0.85 | ✅ |
| Recall | **~0.87** | ≥ 0.75 | ✅ |
| F1-Score | **≥ 0.80** | ≥ 0.80 | ✅ |

**Ví dụ merge thành công:**

| Raw mention | → Canonical | Score |
|-------------|------------|-------|
| "htn" | "hypertension" | Dict |
| "high blood pressure" | "hypertension" | 0.88 |
| "chest discomfort" | "chest pain" | 0.87 |
| "Tylenol" | "acetaminophen" | Dict |

**Ví dụ không merge (đúng):**

| A | B | Score | Vì sao |
|---|---|-------|--------|
| "diabetes" | "diabetes insipidus" | 0.72 | Khác bệnh |
| "pain" | "back pain" | — | Substring filter |

---

## SLIDE 11 — BUILD KG: KẾT QUẢ TÍCH HỢP (7:15–7:45)

**Tiêu đề:** KG v1.0: Tích hợp 3 nguồn thành công

**2 cột:**

**Cột trái — Số liệu KG:**

| | Count |
|--|-------|
| **Tổng nodes** | **18,387** |
| **Tổng relationships** | **117,836** |
| CO_MENTIONED | 74,904 |
| CAUSES | 27,792 |
| TREATED_BY_SPECIALTY | 7,642 |
| HAS_BRAND | 4,154 |
| TREATS | 1,472 |
| HAS_SYMPTOM | 321 |

**Cột phải — Minh chứng tích hợp:**

```
6 bệnh overlap cả 3 nguồn:
  ✅ hypertension  (Bo1 + Bo2 + Bo3)
  ✅ acne          (Bo1 + Bo2 + Bo3)
  ✅ migraine      (Bo1 + Bo2 + Bo3)
  ✅ hypothyroidism(Bo1 + Bo2 + Bo3)
  ✅ pneumonia     (Bo1 + Bo2 + Bo3)
  ✅ psoriasis     (Bo1 + Bo2 + Bo3)
```

**Hình ảnh (quan trọng):**
- Screenshot từ Neo4j Browser: `MATCH (d:Disease) WHERE 'bo1' IN d.source AND 'bo2' IN d.source AND 'bo3' IN d.source RETURN d.name, d.source`
- Thêm node properties panel hiện ra aliases[], source[]

---

## SLIDE 12 — DEMO LIVE 1: CHẨN ĐOÁN FLOW (8:00–9:00)

> **[Đây là slide chuyển sang demo live — không cần nhiều chữ]**

**Tiêu đề:** 🔴 DEMO: Triệu chứng → Bệnh → Thuốc

**Nội dung slide (tối giản):**

```
Câu hỏi: "Bệnh nhân có chest pain → cần làm gì?"

MATCH (s:Symptom {name: 'chest pain'})
      <-[:HAS_SYMPTOM]-(d:Disease)
      <-[:TREATS]-(drug:Drug)
WHERE drug.rating >= 7
RETURN d.name, drug.generic_name, drug.rating
ORDER BY drug.rating DESC LIMIT 10
```

**Expected output hiển thị trên slide (hoặc live):**

| Disease | Drug | Rating |
|---------|------|--------|
| hypertension | betaxolol | 10.0 |
| hypertension | amlodipine | 9.5 |
| pneumonia | lefamulin | 10.0 |

**Thêm dưới slide:** "Thử tiếp: `headache`, `fatigue`"

**Hình ảnh:**
- Screenshot Neo4j Browser với kết quả query *(chụp trước, phòng case demo fail)*

---

## SLIDE 13 — DEMO LIVE 2: HIDDEN RELATIONSHIP (9:00–10:00)

> **[Demo highlight nhất của bài]**

**Tiêu đề:** 🔍 DEMO: Phát hiện mối quan hệ ẩn — Drug Repurposing

**Setup story (3 dòng):**
> "Aspirin không có edge TREATS → hypertension trong dataset structured.  
> Nhưng trong 4,943 ghi chú lâm sàng, aspirin được nhắc cùng hypertension **121 lần**.  
> Đây là tín hiệu off-label use mà dữ liệu structured không capture được."

**Query + kết quả:**

```cypher
MATCH (drug:Drug {generic_name: 'aspirin'})-[co:CO_MENTIONED]-(dis:Disease)
WHERE NOT EXISTS { MATCH (drug)-[:TREATS]->(dis) }
  AND co.weight >= 30
RETURN dis.name, co.weight ORDER BY co.weight DESC LIMIT 10
```

| Disease | CO_MENTIONED weight |
|---------|---------------------|
| pain | 95 |
| **hypertension** | **121** |
| chest pain | 83 |
| edema | 73 |
| bleeding | 71 |

**Hình ảnh (cực kỳ impactful):**
- Mở file `output/viz/03_aspirin_hidden.html` — pyvis graph
- Aspirin ở giữa, xung quanh là diseases CO_MENTIONED (màu xám) + TREATS (màu xanh)
- Screenshot lại để backup

---

## SLIDE 14 — GRAPH ANALYTICS: LOUVAIN (10:00–10:45)

**Tiêu đề:** Community Detection: Louvain tìm ra 3,662 cụm

**Layout: 2 phần**

**Trái — Thống kê:**
- 3,662 communities phát hiện
- Modularity: 0.2588 *(ghi chú: gần 0.3 — ngưỡng "tốt")*
- Graph projection: Drug + Disease + Symptom + Specialty

**Phải — Top 5 cluster có ý nghĩa lâm sàng:**

| Cluster | Size | Chủ đề |
|---------|------|--------|
| 5119 | 1,277 | Nhiễm khuẩn + giảm đau (doxycycline, codeine) |
| 2120 | 978 | **Tim mạch** (aspirin, nitroglycerin, metoprolol, angina) |
| 264 | 635 | Ung thư + tiêu hóa (doxorubicin, vincristine) |
| 3647 | 538 | **ADHD** (amphetamine, methylphenidate) |
| 1234 | 124 | Da liễu / Acne |

**Hình ảnh:**
- Screenshot `output/viz/02_cardio_cluster.html` — cardiovascular cluster
- Mạng lưới dense của các drug tim mạch + diseases liên quan

---

## SLIDE 15 — GRAPH ANALYTICS: CENTRALITY (10:45–11:15)

**Tiêu đề:** Centrality: Node nào quan trọng nhất trong KG?

**2 bảng song song:**

**PageRank — Node "trung tâm" (nhiều kết nối quan trọng):**

| Rank | Node | Label | Score |
|------|------|-------|-------|
| 1 | pain | Disease | 107.69 |
| 2 | hypertension | Disease | 66.64 |
| 3 | bleeding | Disease | 50.21 |
| 4 | dyspnea | Disease | 49.02 |
| 5 | edema | Disease | 45.42 |

**Betweenness — Node "cầu nối" (nằm trên nhiều shortest path):**

| Rank | Node | Score |
|------|------|-------|
| 1 | pain | 517,620 |
| 2 | hypertension | 293,831 |
| 3 | acne | 159,635 |
| 4 | pneumonia | 133,342 |

**Insight (bullet nổi bật):**
> "Pain và hypertension score cao cả 2 metric → vừa là hub kết nối vừa là cầu nối — đây là node quan trọng nhất của toàn bộ KG y khoa"

---

## SLIDE 16 — GRAPH ANALYTICS: LINK PREDICTION (11:15–11:45)

**Tiêu đề:** Link Prediction — Gợi ý Drug Repurposing (BONUS)

**Giải thích ngắn:**
> Adamic-Adar score đo xác suất 2 node sẽ có kết nối dựa trên "bạn chung".  
> Drug-Disease có AA cao nhưng **không có TREATS** → ứng viên off-label.

**Bảng top candidates:**

| Drug | Disease | CO_MENTIONED | Adamic-Adar |
|------|---------|-------------|-------------|
| **aspirin** | **pain** | 95 | **121.63** |
| **aspirin** | **hypertension** | 121 | **116.31** |
| aspirin | chest pain | 83 | 97.86 |
| lasix | dyspnea | 58 | 73.12 |
| insulin | infection | 45 | 61.2 |

**Note ở đáy slide:**
> *Aspirin đã được nghiên cứu trong điều trị cardiovascular events — KG "phát hiện lại" điều này từ dữ liệu lâm sàng, không cần domain knowledge trước.*

---

## SLIDE 17 — VISUALIZATION (11:45–12:30)

**Tiêu đề:** 🎨 Visualization: 4 Interactive Subgraphs (pyvis)

**Layout: 2×2 grid screenshots**

```
┌─────────────────────┐  ┌─────────────────────┐
│  01_hypertension    │  │  02_cardio_cluster   │
│  context.html       │  │  .html               │
│  [SCREENSHOT]       │  │  [SCREENSHOT]        │
│  Drug+Symptom+      │  │  Louvain cluster     │
│  DrugClass+Spec     │  │  tim mạch            │
└─────────────────────┘  └─────────────────────┘
┌─────────────────────┐  ┌─────────────────────┐
│  03_aspirin_        │  │  04_diagnosis_       │
│  hidden.html        │  │  flow.html           │
│  [SCREENSHOT]       │  │  [SCREENSHOT]        │
│  Off-label hints    │  │  headache→drug path  │
└─────────────────────┘  └─────────────────────┘
```

> **Hướng dẫn chụp screenshot:**
> 1. Mở từng file HTML trong Chrome/Edge
> 2. Đợi physics stabilize (~5 giây)
> 3. Zoom/pan để thấy rõ cấu trúc
> 4. F12 → Device toolbar → set 1280×720 → chụp
> 5. Hoặc Windows Snipping Tool (Win+Shift+S)

**Ghi chú demo live:** *"Sẽ mở file trực tiếp trong browser — kéo thả node, hover xem chi tiết"*

---

## SLIDE 18 — DEMO LIVE 3: VISUALIZATION (12:30–13:00)

> **[Slide chuyển tiếp sang demo pyvis — tối giản]**

**Tiêu đề:** 🔴 DEMO: Interactive Graph Visualization

**4 dòng giới thiệu:**
1. `01` — Hypertension: toàn bộ context lâm sàng từ 3 bộ dữ liệu
2. `02` — Cardiovascular cluster: Louvain tự phát hiện nhóm tim mạch
3. `03` — Aspirin hidden: Drug repurposing candidates
4. `04` — Diagnosis flow: headache → migraine → sumatriptan

**Tip demo:** Zoom vào hypertension node, hover xem tooltip, kéo node ra để thấy connections.

---

## SLIDE 19 — TỔNG KẾT KẾT QUẢ (13:00–13:30)

**Tiêu đề:** Kết quả đạt được

**Bảng tổng kết (có màu ✅):**

| Mục tiêu | Target | Kết quả | |
|----------|--------|---------|--|
| Ontology | 7 node + 7 rel | 7 node + 7 rel | ✅ |
| KG scale | > 10,000 nodes | **18,387 nodes** | ✅ |
| Entity Resolution F1 | ≥ 0.80 | **≥ 0.80** | ✅ |
| Heterogeneous integration | 2+ sources | **3 sources tích hợp** | ✅ |
| Graph Analytics | ≥ 3 thuật toán | **4 thuật toán** | ✅ |
| Hidden relationships | Có CO_MENTIONED | **74,904 edges** | ✅ |
| Visualization | Interactive | **4 pyvis HTML** | ✅ |

**Bottom highlight:**
> "6 disease nodes overlap cả 3 nguồn → Minh chứng Entity Resolution hoạt động đúng"

---

## SLIDE 20 — HẠN CHẾ (13:30–13:50)

**Tiêu đề:** Hạn chế & Ghi nhận

**3 bullet ngắn gọn, trung thực:**

1. **NER recall ~70-80%**: scispaCy bỏ sót abbreviation hiếm gặp → một số entity không được capture
2. **Louvain modularity 0.26** (dưới 0.3): KG dense với nhiều CO_MENTIONED → cộng đồng chồng lấn
3. **Gold set 150 mẫu**: Đủ cho đồ án nhưng cần mẫu lớn hơn để đánh giá thống kê mạnh

**Note:** Hạn chế là bình thường — thể hiện sự trung thực và hiểu sâu về kết quả.

---

## SLIDE 21 — HƯỚNG PHÁT TRIỂN (13:50–14:10)

**Tiêu đề:** Hướng phát triển tiếp theo

**5 bullet với icon:**

- 🤖 **Relation Extraction**: Trích xuất quan hệ tường minh từ text (Drug TREATS Disease) thay vì chỉ co-occurrence
- 🧠 **GNN Link Prediction**: Node2Vec + Graph Convolutional Network thay cho Adamic-Adar
- 🌐 **Web UI**: Search interface cho bác sĩ: nhập triệu chứng → KG trả về gợi ý
- 📚 **UMLS CUI mapping**: Ánh xạ entity về chuẩn quốc tế (Unified Medical Language System)
- ⏱️ **Temporal analysis**: Thêm chiều thời gian nếu có longitudinal data

---

## SLIDE 22 — KẾT LUẬN + Q&A (14:10–15:00)

**Tiêu đề:** Kết luận

**3 takeaway lớn (font to, tối giản):**

> **1.** Tích hợp 3 nguồn heterogeneous thành 1 KG thống nhất — 18,387 nodes, 117,836 edges

> **2.** Phát hiện hidden relationship: CO_MENTIONED from NLP reveals drug repurposing candidates không có trong structured data

> **3.** Graph Analytics xác nhận ý nghĩa lâm sàng: clusters tương ứng chuyên khoa, PageRank node y học quan trọng

**Dòng cuối:**
```
🔗 Repository: github.com/[team_repo]
📧 Contact: [email]
```

**Design:** Slide tối giản, chỉ 3 con số lớn + "Thank you / Q&A" ở cuối.

---

## PHỤ LỤC: CHECKLIST CHUẨN BỊ DEMO DAY

### Trước demo (ngày hôm trước)
- [ ] Restore Neo4j backup: `kg_v1_post_fixes.tar.gz`
- [ ] Verify kết nối: `http://localhost:7474` login được
- [ ] Chạy lại: `python notebooks/visualization.py` → kiểm tra 4 HTML files
- [ ] Mở sẵn 4 tab HTML trong Chrome
- [ ] Copy queries từ `docs/sample_queries.cypher` vào Neo4j Browser
- [ ] Chụp screenshot backup cho mọi demo (phòng case lỗi live)

### Trong khi thuyết trình
- [ ] Demo Neo4j Browser (Slide 12, 13): chạy queries trực tiếp
- [ ] Demo pyvis HTML (Slide 18): double-click file, kéo thả node
- [ ] Slide backup screenshots đã chuẩn bị (nếu demo fail)

### Câu hỏi thường gặp
- **"Tại sao modularity chỉ 0.26?"** → Do KG dense: CO_MENTIONED chiếm 64% edges, nhiều singleton node. Có thể cải thiện bằng subgraph filtering hoặc loại bỏ CO_MENTIONED khi cluster.
- **"ER chỉ 684 real merges / 8,017 rows?"** → 7,321 mapping còn lại là cosmetic (chỉ khác dấu câu/case) — build_kg.py đã filter đúng, không gây lỗi.
- **"Tại sao không dùng UMLS?"** → Ngoài scope đồ án, UMLS yêu cầu license và cài đặt phức tạp.
- **"Drug repurposing có đáng tin không?"** → Đây là gợi ý từ co-occurrence — cần clinical validation. Aspirin/cardiovascular là case nổi tiếng đã được nghiên cứu y khoa.

---

## GỢI Ý HÌNH ẢNH CẦN CHỤP / TẠO

| Hình | Nguồn | Slide |
|------|-------|-------|
| Screenshot Neo4j Browser — query chest pain results | Chạy live hoặc chụp sẵn | 12 |
| Screenshot Neo4j Browser — 6 diseases overlap 3 sources | Chạy query MATCH (d:Disease) WHERE 'bo1' IN d.source... | 11 |
| Screenshot pyvis 01_hypertension_context.html | Mở file HTML | 17 |
| Screenshot pyvis 02_cardio_cluster.html | Mở file HTML | 17 |
| Screenshot pyvis 03_aspirin_hidden.html | Mở file HTML | 13, 17 |
| Screenshot pyvis 04_diagnosis_flow.html | Mở file HTML | 17 |
| Screenshot entity properties panel (aliases, source[]) | Neo4j Browser click vào node | 11 |
| Diagram 3 data sources → KG (PowerPoint shape) | Tự vẽ | 2 |
| Pipeline flow diagram | Tự vẽ trong PowerPoint | 6 |
| Ontology diagram (đẹp hơn text) | draw.io hoặc PowerPoint | 7 |
