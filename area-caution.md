# Area: "Caution" list

Areas that may need alignment with business: city-states, non-standard administrative structure (continent→country→province→city), or very small territories. Brief verification as of March 2025. Use this list when discussing coverage and hierarchy with the business team.

---

## Asia

| Area | Verification | Notes |
|------|---------------|-------|
| **Singapore** | ✅ Verified | Unitary state **with no provinces/states**. Has 5 regions and planning areas for planning/census, not official admin levels. In our model: **country only**, no children. |
| **Hong Kong** | ✅ Verified | **SAR of China**, not a sovereign country. Divided into **18 districts** (Islands, Kwai Tsing, North, Sai Kung, etc.). If treated as a separate "area", structure is district-based, not province→city. |
| **Macau** | ✅ Verified | **SAR of China**. Formerly 2 municipalities + 7 parishes; municipalities abolished in 2001. Parishes are now largely symbolic. Very small; use caution. |
| **Brunei** | ✅ Verified | Has **4 districts** (daerah): Belait, Brunei-Muara, Temburong, Tutong. Districts are subdivided into mukims. Small, "Singapore-like" in scale, but has **one level** below country (district). |
| **Maldives** | ✅ Verified | **Atoll system**: ~20 administrative atolls (Haa Alifu, Shaviyani, Raa, Baa, etc.), not classic province/city. Naming and structure differ. |

---

## Middle East

| Area | Verification | Notes |
|------|---------------|-------|
| **Qatar** | ✅ Verified | **8 municipalities** (baladiyat): Doha, Al Rayyan, Al Wakrah, Al Khor, etc. Zones below. City-state / one level below country. |
| **Bahrain** | ✅ Verified | **4 governorates**: Capital, Northern, Southern, Muharraq. One level below country. |
| **Kuwait** | ✅ Verified | **6 governorates**: Ahmadi, Asimah (Capital), Hawalli, Jahra, Farwaniya, Mubarak Al-Kabeer. One level below country. |

---

## Europe (microstates)

| Area | Verification | Notes |
|------|---------------|-------|
| **Monaco** | ✅ Verified | City-state, ~2 km². No formal territorial subdivisions (quartiers exist for statistics). **Country only** in our model. |
| **Vatican** | ✅ Verified | **No administrative subdivisions**; ~0.44 km². Governatorate/directorates are for government functions, not geographic divisions. **Country only**. |
| **San Marino** | ✅ Verified | **9 castelli** (municipalities): Acquaviva, Borgo Maggiore, City of San Marino, Serravalle, etc. Curazie below. Has one level below country. |

---

## Summary

- **Country only (no children):** Singapore, Monaco, Vatican.
- **One level below country** (district / governorate / municipality / atoll): Hong Kong (18 districts), Macau (symbolic parishes), Brunei (4 districts), Maldives (atolls), Qatar (8 municipalities), Bahrain (4 governorates), Kuwait (6 governorates), San Marino (9 castelli).
- **Note:** Hong Kong and Macau are **not sovereign countries** (SARs of China). If area master data uses "country" = sovereign state only, they may be treated as special entities or under China. For dropdown/navigation, still use caution as their structure does not follow province→city.

All entries on this caution list are **verified** — the structures above match common sources (Wikipedia, Statoids, CIA Factbook, government sites).
