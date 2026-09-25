# Hub Supply Engine — contoh data & naratif

Catatan kerja, **bukan TD**.  
**DDL:** [hub_supply_engine.sql](../SQL/hub_supply_engine.sql).

Dokumen ini pakai **ID fiktif** (prefix mudah dibaca) supaya tabel bisa dilacak antar-bab. Angka dalam **IDR** kecuali disebut lain.

---

## Konteks trip (pemain tetap)

| Entitas | ID / kode | Keterangan |
| :--- | :--- | :--- |
| Product trip | `product-trip-7f2a` | `tour_trips` — keberangkatan 12 Jun 2026 |
| Hub trip | `hub-trip-7f2a` | `hub.trips` — 1:1 dengan product |
| Kode paket | **GWE-TYO-12JUN26** | 25 pax, Tokyo 5D4N |
| Mata uang reporting | **IDR** | `planned_total_base` / `committed_total_base` |

Semua baris di bawah **`trip_id = hub-trip-7f2a`** kecuali dinyatakan lain.

---

## Bab 1 — Planner mengisi BOM (hanya PLAN)

**Januari 2026.** Tim product/Ops planner menyiapkan **rencana HPP** belum ada telepon ke vendor, belum ada PNR/conf.

### `hub.trip_plan_items`

| id | seq | item_kind | title | planned_qty | unit_type | planned_total_base | fulfill_status |
| :--- | ---: | :--- | :--- | ---: | :--- | ---: | :--- |
| `plan-01` | 1 | FLIGHT | Block seat GA 875 CGK-NRT return | 25 | PER_PAX | 375.000.000 | UNFULFILLED |
| `plan-02` | 2 | HOTEL | Hotel Shinjuku 4★ — 10 kamar × 3 malam | 30 | PER_NIGHT | 60.000.000 | UNFULFILLED |
| `plan-03` | 3 | TRANSFER | Bus airport ↔ hotel (1 unit 40 seat) | 1 | PER_UNIT | 8.000.000 | UNFULFILLED |
| `plan-04` | 4 | TIPPING | Tip guide + driver (budget) | 1 | PER_TRIP | 5.000.000 | UNFULFILLED |
| `plan-05` | 5 | ATTRACTION | Disney ticket group | 25 | PER_PAX | 37.500.000 | UNFULFILLED |

### `hub.trip_plan_item_flights` (hanya `plan-01`)

| plan_item_id | flight_mode | seats | price_per_pax | deposit_per_seat | planned (implicit) |
| :--- | :--- | ---: | ---: | ---: | :--- |
| `plan-01` | GIT | 25 | 15.000.000 | 2.000.000 | 375 jt total block |

**Narasi:** Di UI tab **Plan**, Bos melihat **total budget HPP rencana ≈ Rp 485,5 juta** (jumlah baris di atas). Belum ada baris di `fulfillments` — ini **kertas kerja**, bukan kenyataan vendor.

---

## Bab 2 — Hotel Tokyo: satu rencana, dua vendor (split fulfill)

**Februari 2026.** Ops booking hotel. Agoda B2B cuma punya **8 kamar**; **2 kamar** dipesan langsung ke hotel (lebih mahal).

### `hub.fulfillments`

| id | seq | fulfillment_kind | coverage | supplier_name | vendor_reference | status | committed_total_base |
| :--- | ---: | :--- | :--- | :--- | :--- | :--- | ---: |
| `ful-01` | 1 | HOTEL | PLANNED | Agoda B2B | `AGD-TYO-8821` | CONFIRMED | 48.000.000 |
| `ful-02` | 2 | HOTEL | PLANNED | Hotel Shinjuku Direct | `HTL-SJK-4410` | CONFIRMED | 15.000.000 |

### `hub.fulfillment_lines`

| id | fulfillment_id | plan_item_id | fulfilled_qty | allocated_cost_base | Catatan |
| :--- | :--- | :--- | ---: | ---: | :--- |
| `fl-01` | `ful-01` | `plan-02` | 24 | 48.000.000 | 8 kamar × 3 malam = 24 room-nights |
| `fl-02` | `ful-02` | `plan-02` | 6 | 15.000.000 | 2 kamar × 3 malam = 6 room-nights |

### Update plan (hasil rollup BE)

| plan_item_id | fulfill_status | Alasan |
| :--- | :--- | :--- |
| `plan-02` | **FULFILLED** | 24 + 6 = **30** room-nights = `planned_qty` |

### `hub.fulfillment_detail_fields` (cuplikan fulfill `ful-02`)

| fulfillment_id | field_key | field_value |
| :--- | :--- | :--- |
| `ful-02` | `hotel.phone` | +81-3-1234-5678 |
| `ful-02` | `hotel.address` | 2-1-1 Shinjuku, Tokyo |

**Narasi:** **Satu baris plan** hotel tetap utuh sebagai **budget Januari (60 jt)**. Lapangan **dua surat jalan vendor** → dua header `fulfillments`. Variance **committed vs budget** untuk `plan-02`:

| Metrik | Nilai |
| :--- | ---: |
| Budget (plan) | 60.000.000 |
| Committed (Σ allocated pada plan-02) | **63.000.000** |
| Selisih | **−3.000.000** (overbudget deal) |

Checklist **VERIFY** hotel → `status = CONFIRMED`, `confirmed_at` terisi; **uang belum keluar** sampai Bab 6.

---

## Bab 3 — Flight GIT: plan komersial + PNR + segments

**Februari 2026.** Ops block seat GA; PNR keluar; jadwal paste dari GDS.

### `hub.fulfillments`

| id | seq | fulfillment_kind | coverage | vendor_reference | status | committed_total_base |
| :--- | ---: | :--- | :--- | :--- | :--- | ---: |
| `ful-10` | 3 | FLIGHT | PLANNED | `7V9VKN` | BOOKED | 375.000.000 |

### `hub.fulfillment_lines`

| fulfillment_id | plan_item_id | fulfilled_qty | allocated_cost_base |
| :--- | :--- | ---: | ---: |
| `ful-10` | `plan-01` | 25 | 375.000.000 |

### `hub.fulfillment_flight_segments` (replace-set)

| fulfillment_id | seq | airline | flight | org | dest | departure_at (UTC+9) |
| :--- | ---: | :--- | :--- | :--- | :--- | :--- |
| `ful-10` | 1 | GA | 875 | CGK | NRT | 2026-06-12 06:10 |
| `ful-10` | 2 | GA | 876 | NRT | CGK | 2026-06-16 18:30 |

**Narasi:** **Uang & seat** tetap di **plan** (`trip_plan_item_flights`). **PNR & jadwal** hidup di **fulfill** — bukan di plan. Reschedule Juni (TK time change): **PNR sama**, segments di-replace; `ful-10.id` stabil → checklist deposit/issued tidak putus.

| plan-01 fulfill_status | FULFILLED (qty pax ter-cover 25/25) |

---

## Bab 4 — BUNDLED: rute tanpa pembelian seat terpisah

Planner tambah **penerbangan sudah termasuk paket land** (wholesale sudah bayar tiket).

### Plan tambahan

| id | item_kind | title | planned_total_base | fulfill_status |
| :--- | :--- | :--- | ---: | :--- |
| `plan-06` | FLIGHT | Info rute — included in DMC package | **0** | UNFULFILLED |

### Fulfill (BUNDLED-style)

| id | fulfillment_kind | coverage | vendor_reference | status | committed_total_base |
| :--- | :--- | :--- | :--- | :--- | ---: |
| `ful-11` | FLIGHT | PLANNED | *(NULL)* | DRAFT | 0 |

Segments diisi untuk **papan keberangkatan**; tidak ada Request Payment dari fulfill ini. HPP tiket sudah di baris **WHOLESALE** (contoh di Bab 7).

---

## Bab 5 — Bus 40 seat rencana → 2× bus 20 seat (split fulfill)

**Maret 2026.** Vendor transfer bilang bus 40 seat habis; dikirim **2 bus 20 seat** (beda polisi).

### Fulfillments

| id | fulfillment_kind | coverage | vendor_reference | status | committed_total_base |
| :--- | :--- | :--- | :--- | :--- | ---: |
| `ful-20` | TRANSFER | PLANNED | `TRF-BUS-A` | CONFIRMED | 4.200.000 |
| `ful-21` | TRANSFER | PLANNED | `TRF-BUS-B` | CONFIRMED | 4.500.000 |

### Fulfillment lines (keduanya ke **plan-03**)

| fulfillment_id | plan_item_id | fulfilled_qty | allocated_cost_base |
| :--- | :--- | ---: | ---: |
| `ful-20` | `plan-03` | 0.5 | 4.200.000 |
| `ful-21` | `plan-03` | 0.5 | 4.500.000 |

*(Policy contoh: `planned_qty = 1` PER_UNIT bus; **fulfilled_qty** pecah proporsi 0,5 + 0,5 = 1 unit terpenuhi — BE bisa juga pakai qty=1+1 dengan unit PER_UNIT dan flag over-unit; yang penting **M:N mental**.)*

### Detail fields

| fulfillment_id | field_key | field_value |
| :--- | :--- | :--- |
| `ful-20` | `transfer.plate_no` | B 1234 XYZ |
| `ful-21` | `transfer.plate_no` | B 5678 XYZ |

| plan-03 fulfill_status | FULFILLED |
| Committed vs budget 8 jt | **8,7 jt** (+700 rb) |

---

## Bab 6 — Satu invoice DMC, dua baris plan (merge fulfill)

DMC **Sakura Travel** mengirim **satu invoice** untuk **land wholesale + visa grup** (dekat ke lapangan).

### Plan (sudah ada dari Bab 1)

| id | item_kind | title | planned_total_base |
| :--- | :--- | ---: |
| `plan-07` | WHOLESALE_PACKAGE | Land + meal DMC Tokyo | 120.000.000 |
| `plan-08` | VISA | Visa grup Jepang | 12.500.000 |

### Satu fulfillment, dua lines

| id | coverage | supplier_name | vendor_reference | status | committed_total_base |
| :--- | :--- | :--- | :--- | :--- | ---: |
| `ful-30` | **MIXED** | Sakura Travel | `DMC-2026-0192` | BOOKED | 132.500.000 |

| fulfillment_id | plan_item_id | fulfilled_qty | allocated_cost_base |
| :--- | :--- | ---: | ---: |
| `ful-30` | `plan-07` | 1 | 120.000.000 |
| `ful-30` | `plan-08` | 25 | 12.500.000 |

**Narasi:** **Satu PNR/invoice vendor** → **satu header fulfill**; alokasi ke **dua plan item**. Finance PO/bill bisa satu dokumen dengan `fulfillment_id = ful-30`; variance per plan item dihitung dari **allocated**, bukan dari header dobel.

---

## Bab 7 — Tipping: lewat plan vs langsung expense

### 7A — Path plan → fulfill (terencana)

| Plan `plan-04` | Budget 5 jt TIPPING |
| Fulfill `ful-40` | Vendor TL Tokyo, ref `TIP-TL-01`, committed 5 jt, CONFIRMED |
| Line | `plan-04` ← 100% allocated 5 jt |

Ops **Request Payment** → Finance bill → event ke Hub:

### `hub.fulfillment_cash_events` (cuplikan)

| fulfillment_id | finance_doc_kind | finance_doc_id | event_kind | paid_amount_base | occurred_at |
| :--- | :--- | :--- | :--- | ---: | :--- |
| `ful-40` | VENDOR_BILL | `fin-bill-901` | SETTLEMENT | 5.000.000 | 2026-04-10 |

### 7B — Path langsung expense (tanpa plan)

**Di lapangan**, TL bayar tip parkir cash **Rp 350.000** — tidak sempat buat plan.

Finance posting:

| Tabel finance | `expenses.id = fin-exp-772`, `trip_id = hub-trip-7f2a`, POSTED |

Hub ingest (read model):

### `hub.trip_cost_links`

| trip_id | finance_doc_kind | finance_doc_id | plan_item_id | title_snapshot | amount_base | occurred_at |
| :--- | :--- | :--- | :--- | :--- | ---: | :--- |
| `hub-trip-7f2a` | EXPENSE | `fin-exp-772` | *(NULL)* | Tip parkir cash TL | 350.000 | 2026-06-13 |

**Narasi:** **Tipping bisa keduanya.** Dashboard trip: budget tip **5 jt** (plan) + **350 rb** (direct cost) = **realized tip spend** terpisah dari committed fulfill `ful-40` sampai bill lunas.

---

## Bab 8 — Fulfill tanpa plan (unplanned)

**Di Tokyo**, bus parah; Ops sewa bus pengganti **tanpa baris plan**.

### `hub.fulfillments`

| id | coverage | fulfillment_kind | vendor_reference | status | committed_total_base |
| :--- | :--- | :--- | :--- | :--- | ---: |
| `ful-50` | **UNPLANNED** | TRANSFER | `EMRG-BUS-01` | BOOKED | 6.000.000 |

### `hub.fulfillment_lines`

| fulfillment_id | plan_item_id | fulfilled_qty | allocated_cost_base |
| :--- | :--- | ---: | ---: |
| `ful-50` | **NULL** | 1 | 6.000.000 |

**Narasi:** **Common** untuk emergency. Trip P&L: **+6 jt committed** tanpa budget plan sebelumnya. Ops bisa later **menambah plan retroaktif** (policy produk) atau biarkan murni **unplanned cost**. Finance tetap bayar lewat bill → `fulfillment_cash_events`.

---

## Bab 9 — Realisasi uang (timeline ringkas)

Gabungan event **fulfill path** + **direct cost** (contoh seleksi):

| Tanggal | Sumber | ID | Jenis | paid_amount_base | Keterangan |
| :--- | :--- | :--- | :--- | ---: | :--- |
| 2026-03-01 | fulfill | `ful-10` | ADVANCE | 50.000.000 | DP seat GA (25×2 jt) |
| 2026-04-10 | fulfill | `ful-40` | SETTLEMENT | 5.000.000 | Tip TL bill |
| 2026-04-15 | fulfill | `ful-01` | SETTLEMENT | 48.000.000 | Pelunasan Agoda |
| 2026-06-13 | trip_cost | `fin-exp-772` | EXPENSE | 350.000 | Tip cash |
| 2026-06-20 | fulfill | `ful-10` | SETTLEMENT | 325.000.000 | Pelunasan block seat |

**Narasi:** Hub **tidak** jadi SoT pembayaran — hanya **mirror** untuk Ops/Bos. SoT tetap **finance**.

---

## Bab 10 — Cancel fulfill: plan kembali partial

Hotel Direct (`ful-02`) **dibatalkan** vendor; 6 room-nights harus dicari lagi.

| Perubahan | Nilai |
| :--- | :--- |
| `ful-02.status` | **CANCELLED** |
| Lines `fl-02` | diabaikan rollup (policy: exclude cancelled fulfill) |
| `plan-02.fulfill_status` | **PARTIAL** (24/30 room-nights dari Agoda saja) |

**Narasi:** **Plan 60 jt tidak dihapus** — masih target. Ops buat **`ful-03`** baru ke vendor lain untuk 6 room-nights sisa tanpa mengedit history `ful-02`.

---

## Ringkasan dashboard trip (contoh angka)

Per **plan item** (committed dari lines, paid = alokasi sederhana):

| plan | Budget | Committed | Paid (≈) | Catatan |
| :--- | ---: | ---: | ---: | :--- |
| plan-01 FLIGHT | 375 jt | 375 jt | 375 jt | DP + pelunasan |
| plan-02 HOTEL | 60 jt | 63 jt → 48 jt* | 48 jt | *setelah cancel Direct |
| plan-03 TRANSFER | 8 jt | 8,7 jt | 0 | belum bill |
| plan-04 TIPPING | 5 jt | 5 jt | 5 jt | |
| plan-05 ATTRACTION | 37,5 jt | 0 | 0 | belum booking |

**Di luar plan:**

| Sumber | Committed / cost |
| :--- | ---: |
| `ful-50` UNPLANNED | 6 jt committed |
| `trip_cost_links` expense | 350 rb paid |

---

## Cheat sheet relasi

```text
trip_plan_items (1) ──< fulfillment_lines >── (N) fulfillments
fulfillments (1) ──< fulfillment_flight_segments
fulfillments (1) ──< fulfillment_detail_fields
fulfillments (1) ──< fulfillment_cash_events     ← mirror finance
hub.trips (1) ──< trip_cost_links                 ← expense langsung
```

---

## Lihat juga

- [hub_supply_engine.sql](../SQL/hub_supply_engine.sql) — DDL
- [hub_plan_lines_and_flights.md](./hub_plan_lines_and_flights.md) — prep checklist/finance seed (legacy naming)
- [lite_erp.md](./lite_erp.md) — SoT expense & payment
