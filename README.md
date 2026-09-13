<div align="center">

# 🗺️ Yandex Maps SKILL

![Next.js](https://img.shields.io/badge/Next.js-15-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![Yandex Maps](https://img.shields.io/badge/Yandex_Maps-v3-FFCC00?style=for-the-badge&logo=yandex&logoColor=black)
![License](https://img.shields.io/badge/License-MIT-lightgrey?style=for-the-badge)

**Copy-paste reference for integrating Yandex Maps JS API v3 into Next.js projects**

</div>

> Production-tested patterns for Yandex Maps v3: script loading, map init, markers, zoom controls, and sidebar interplay. Built from real debugging on a live 700+ venue map.

---

## 💡 Concept

Yandex Maps v3 is poorly documented and full of gotchas: `panTo()` is v2, built-in zoom controls don't exist, Strict Mode double-fires effects, and smooth-scroll libraries hijack map wheel events. This guide is a single-file copy-paste reference — every snippet was debugged and verified on a production site with 714 venue markers across 103 cities.

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| **Script loading** | Idempotent API script injection, safe for React Strict Mode |
| **Map init** | Double-fire guard, teardown cleanup |
| **Behaviors** | Explicit `drag`, `scrollZoom`, `pinchZoom`, `dblClick` |
| **Navigation** | `map.update()` — v3 API, not `panTo()` |
| **Markers** | Imperative `YMapMarker` creation with ref-tracked lifecycle |
| **Zoom controls** | Custom +/- buttons (v3 has no built-in zoom UI) |
| **Sidebar interplay** | `data-lenis-prevent` to stop smooth-scroll stealing wheel events |
| **Layout** | Framed map below viewport height with rounded borders |
| **Pitfalls** | 7 common mistakes catalogued from production debugging |

---

## 🚀 Quick Start

```bash
git clone https://github.com/maximosovsky/yandex-maps-SKILL.git
```

Open `README.md` and copy the snippets into your Next.js project. Each section is a standalone code block.

<details>
<summary>⚙️ Prerequisites</summary>

- Next.js 14+ with App Router
- Yandex Maps API key (get at [developer.tech.yandex.ru](https://developer.tech.yandex.ru/))
- `NEXT_PUBLIC_YANDEX_MAPS_API_KEY` in `.env.local`

</details>

---

## 🏗️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 15 (App Router) |
| Language | TypeScript |
| Maps | Yandex Maps JS API v3 |
| Scroll | ReactLenis (interplay patterns apply to any smooth-scroll lib) |

---

## 🗺️ Roadmap

- [x] Script loading + map init
- [x] Markers + re-rendering
- [x] Zoom controls (custom +/-)
- [x] Sidebar + map wheel interplay
- [x] Layout patterns
- [x] Pitfalls catalog
- [ ] Clusterer plugin example
- [ ] Search suggest via Yandex Geocoder

---

## 🤝 Contributing

Fork → `feature/name` → PR

Found a pitfall not listed here? Add it.

---

## 📄 License

[Maxim Osovsky](https://www.linkedin.com/in/osovsky/). Licensed under [MIT](https://opensource.org/licenses/MIT).