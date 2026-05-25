# Spec Frontend — Hiển thị bảng (Table) trong Text Comparison Tool

## Tổng quan

Tool so sánh 2 file Word (ORIGINAL vs MODIFIED), hiển thị sự thay đổi của bảng theo từng trường hợp.

---

## 1. Logic cấp bảng

| Trường hợp | ORIGINAL (trái) | MODIFIED (phải) |
|---|---|---|
| Bảng bị **xóa** khỏi file 1 | ✅ Hiện toàn bộ bảng | ❌ Trống — label "(đã xóa)" |
| Bảng được **thêm** vào file 2 | ❌ Trống — label "(chưa tồn tại)" | ✅ Hiện toàn bộ bảng |
| Bảng **thay đổi nhiều** (`row_modified`) | colspan 2 — render RowModifiedView theo logical rows | ← |
| Bảng **thay đổi ít** (`full_table`) | ✅ Hiện bảng gốc | ✅ Hiện bảng mới |
| Bảng **không đổi** | ❌ Không hiển thị | ❌ Không hiển thị |

---

## 2. RowModifiedView — Logic từng dòng

Khi `render_mode = row_modified`, render từng dòng theo trạng thái:

| Trường hợp dòng | ORIGINAL (trái) | MODIFIED (phải) |
|---|---|---|
| Dòng bị **xóa** | ✅ Hiện full nội dung dòng, highlight đỏ toàn dòng | ❌ Trống |
| Dòng được **thêm** | ❌ Trống | ✅ Hiện full nội dung dòng, highlight xanh toàn dòng |
| Dòng **có thay đổi nội dung** | ✅ Hiện full nội dung cell | ✅ Hiện full nội dung cell |

> **Quan trọng:** "Full nội dung" bao gồm tất cả các loại nội dung trong cell: text, ảnh, nested table, v.v. — không được bỏ sót.

---

## 3. Nội dung cell — Chi tiết render

| Loại nội dung | Render |
|---|---|
| **Text có thay đổi** | Word-level diff: chữ xóa dùng `.del-span` (đỏ, gạch ngang) — chữ thêm dùng `.ins-span` (xanh, nền) |
| **Text không đổi** | Hiện bình thường |
| **Ảnh** | Hiện ảnh nguyên bản |
| **Nested table** | Hiện nested table nguyên bản như bảng gốc — không diff, không highlight |

---

## 4. Các file liên quan

| File | Vai trò |
|---|---|
| `TableChange.jsx` | Nhận `change` object từ backend, phân loại theo `render_mode`, điều phối render |
| `LogicalFullTable.jsx` | Render 1 bảng đầy đủ từ flat cells (`full_table_original` / `full_table_modified`) |
| `RowModifiedView.jsx` | Render từng dòng theo logical rows khi `render_mode = row_modified` |
| `CellContent.jsx` | Render nội dung 1 cell: text diff, ảnh, nested table |
| `compare.css` | Layout 2 cột ORIGINAL \| MODIFIED side-by-side |

---

## 5. Mapping render_mode → Component

```
render_mode = "table_deleted"      → TableChange → LogicalFullTable (left only)
render_mode = "table_added"        → TableChange → LogicalFullTable (right only)
render_mode = "full_table"         → TableChange → LogicalFullTable (cả 2 side-by-side)
render_mode = "table_layout_changed" → TableChange → LogicalFullTable (cả 2) + badge
render_mode = "row_modified"       → TableChange → RowModifiedView (colspan 2)
```

---

## 6. Bug hiện tại cần fix

- **Case `table_deleted`:** Bảng đang render bên MODIFIED thay vì ORIGINAL.
- **Nguyên nhân khả năng cao:** `change_kind` hoặc `render_mode` từ backend không match đúng điều kiện trong `TableChange.jsx`, rơi xuống case `full_table` — `leftTableData` null còn `rightTableData` có data nên bảng hiện bên phải.

---

## 7. CSS classes tham chiếu

| Class | Ý nghĩa |
|---|---|
| `.del-span` | Text bị xóa: nền đỏ, gạch ngang |
| `.ins-span` | Text được thêm: nền xanh |
| `.row-deleted` | Border đỏ toàn dòng |
| `.row-added` | Border xanh toàn dòng |
| `.row-modified` | Border vàng toàn dòng |
| `.cell-changed` | Nền vàng cell có thay đổi |
| `.cell-added` | Nền xanh cell được thêm |
| `.cell-deleted` | Nền đỏ cell bị xóa |
| `.tbl-side-wrap` | Wrapper bảng trong 1 cột |
| `.change-cell.left` | Cột ORIGINAL |
| `.change-cell.right` | Cột MODIFIED |
 Trường hợp : File 1 có nested table nhưng file 2 (không còn nested table) bị chia thành nhiều dòng biểu diễn các dòng trong nested table kia thì render dòng đó cùng các dòng kia , k cần diff 