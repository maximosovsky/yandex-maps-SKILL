---
name: yandex-maps
description: "Use when adding Yandex Maps v3 to Next.js."
version: 1.0.0
author: Hermes Agent
---

# Yandex Maps JS API v3 in Next.js

## Script loading

Load the API script once, idempotently, with a data attribute so Strict Mode double-mounts don't re-inject:

```tsx
useEffect(() => {
  if (typeof window === 'undefined' || !mapContainerRef.current) return;
  if ((window as any).ymaps3) { setMapLoaded(true); return; }

  const existing = document.querySelector<HTMLScriptElement>('script[data-yandex-maps="v3"]');
  const script = existing || document.createElement('script');
  const onLoad = () => setMapLoaded(true);
  const onError = () => console.error('Yandex Maps API failed to load');
  script.addEventListener('load', onLoad);
  script.addEventListener('error', onError);
  if (!existing) {
    script.dataset.yandexMaps = 'v3';
    script.src = `https://api-maps.yandex.ru/v3/?apikey=${apiKey}&lang=ru_RU`;
    script.async = true;
    document.head.appendChild(script);
  }
  return () => {
    script.removeEventListener('load', onLoad);
    script.removeEventListener('error', onError);
  };
}, []);
```

## Map initialisation

Guard against double-init (React Strict Mode fires effects twice):

```tsx
useEffect(() => {
  if (!mapLoaded || !mapContainerRef.current || typeof window === 'undefined') return;
  if (mapInstanceRef.current) return;
  const ymaps3 = (window as any).ymaps3;
  if (!ymaps3) return;

  let cancelled = false;
  ymaps3.ready.then(() => {
    if (cancelled || !mapContainerRef.current || mapInstanceRef.current) return;
    ymapsRef.current = ymaps3;
    const { YMap, YMapDefaultSchemeLayer, YMapDefaultFeaturesLayer } = ymaps3;
    const map = new YMap(mapContainerRef.current, {
      location: { center: [37.64, 55.76], zoom: 5 },
      behaviors: ['drag', 'scrollZoom', 'pinchZoom', 'dblClick'],
    });
    map.addChild(new YMapDefaultSchemeLayer({}));
    map.addChild(new YMapDefaultFeaturesLayer({}));
    mapInstanceRef.current = map;
    setMapReady(true);
  }).catch((err) => console.error('Yandex Maps init error:', err));

  return () => {
    cancelled = true;
    mapInstanceRef.current?.destroy();
    mapInstanceRef.current = null;
  };
}, [mapLoaded]);
```

## Behaviors

Set explicit `behaviors` in the YMap constructor:

```tsx
behaviors: ['drag', 'scrollZoom', 'pinchZoom', 'dblClick']
```

- `drag` — pan
- `scrollZoom` — mouse wheel zoom
- `pinchZoom` — touch pinch
- `dblClick` — double-click zoom

## Navigation (center / zoom)

v3 uses `map.update()`, NOT `panTo()`:

```tsx
map.update({ location: { center: [lng, lat], zoom: 12 } });                     // instant
map.update({ location: { center: [lng, lat], zoom: 14, duration: 800 } });       // animated
map.update({ location: { zoom: zoomRef.current, duration: 200 } });               // zoom only
```

## Markers

Create imperatively with `YMapMarker`. Track in a ref to remove before re-rendering:

```tsx
const renderMarkers = (items, ymaps3, map) => {
  const { YMapMarker } = ymaps3;
  placemarksRef.current.forEach(pm => map.removeChild(pm));
  placemarksRef.current = [];

  items.forEach(item => {
    const el = document.createElement('div');
    el.innerHTML = `<div style="width:20px;height:20px;border-radius:50%;background:${color};border:2px solid white;box-shadow:0 2px 6px rgba(0,0,0,0.3);cursor:pointer;"></div>`;
    el.onclick = () => { setSelected(item); map.update({ location: { center: [item.lng, item.lat], zoom: 14 } }); };
    const marker = new YMapMarker({ coordinates: [item.lng, item.lat], draggable: false }, el);
    map.addChild(marker);
    placemarksRef.current.push(marker);
  });
};
```

## Custom zoom controls

Yandex Maps v3 doesn't render built-in +/- buttons:

```tsx
const zoomRef = useRef(5);

const changeZoom = (delta: number) => {
  if (!mapInstanceRef.current) return;
  zoomRef.current = Math.min(19, Math.max(2, zoomRef.current + delta));
  mapInstanceRef.current.update({ location: { zoom: zoomRef.current, duration: 200 } });
};
```

Update `zoomRef.current` in every handler that changes zoom.

## Sidebar + map interplay

When the page uses `ReactLenis`: add `data-lenis-prevent` on both panels.

```tsx
<section data-lenis-prevent className="overflow-y-auto overscroll-contain …">list</section>
<section data-lenis-prevent className="overflow-hidden …"><div ref={mapContainerRef} /></section>
```

## Layout

Frame the map below viewport height:

```tsx
<main className="mt-24 mb-6 px-4 flex flex-col sm:flex-row h-[calc(100dvh-7.5rem)] gap-4 overflow-hidden">
  <section className="sm:w-[420px] rounded-2xl border overflow-y-auto …">list</section>
  <section className="sm:flex-grow rounded-2xl border shadow-lg overflow-hidden …">map</section>
</main>
```

## Pitfalls

- `panTo()` is v2 — use `map.update({ location: … })`.
- `if (mapInstanceRef.current) return` — Strict Mode double-fires effects.
- No built-in zoom controls in v3 — add custom +/-.
- `controls: ['zoomControl']` silently fails in v3 — use `behaviors`.
- Smooth-scroll libraries hijack map wheel — use `data-lenis-prevent`.
- `zoomRef` must stay in sync across all zoom-changing handlers.
- Remove old markers before re-rendering — accumulate in ref, `removeChild` each.