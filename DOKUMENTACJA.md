# 🛩️ The Aviator — Dokumentacja kodu i wyjaśnienie Three.js

## Spis treści

1. [Przegląd projektu](#1-przegląd-projektu)
2. [Struktura plików](#2-struktura-plików)
3. [Czym jest Three.js?](#3-czym-jest-threejs)
4. [Krok 1 — Podstawowa scena (main_step1.js)](#4-krok-1--podstawowa-scena-main_step1js)
5. [Krok 2 — Rozbudowana scena (main_step2.js)](#5-krok-2--rozbudowana-scena-main_step2js)
6. [Pełna gra (game.js)](#6-pełna-gra-gamejs)
7. [Jak Three.js renderuje scenę — podsumowanie](#7-jak-threejs-renderuje-scenę--podsumowanie)
8. [Paleta kolorów](#8-paleta-kolorów)
9. [Użyte biblioteki](#9-użyte-biblioteki)

---

## 1. Przegląd projektu

**The Aviator** to gra 3D napisana w JavaScript z użyciem biblioteki **Three.js**. Gracz steruje samolotem za pomocą myszy (lub dotyku na urządzeniach mobilnych), zbiera niebieskie monety (które dodają energię) i unika czerwonych przeszkód. Gra ma system poziomów, energii i dystansu.

Projekt pochodzi z tutoriala Codrops autorstwa **Karima Maaloul** i jest podzielony na 3 etapy:
- **Part 1** (`part1.html`) — Podstawowa scena 3D z samolotem, morzem i chmurami
- **Part 2** (`part2.html`) — Rozbudowana scena z pilotem, falami i animacjami
- **Gra** (`index.html`) — Pełna gra z monetami, wrogami, energią i poziomami

---

## 2. Struktura plików

```
TheAviator/
├── index.html          ← Pełna gra
├── part1.html          ← Tutorial krok 1
├── part2.html          ← Tutorial krok 2
├── css/
│   ├── demo.css        ← Nawigacja, linki, fonty
│   ├── styles.css      ← Prosty styl tła (Part 1 & 2)
│   └── game.css        ← UI gry (wynik, energia, nagłówek)
├── js/
│   ├── three.js        ← Biblioteka Three.js r75 (silnik 3D)
│   ├── TweenMax.js     ← GreenSock TweenMax v1.17 (animacje)
│   ├── main_step1.js   ← Kod kroku 1
│   ├── main_step2.js   ← Kod kroku 2
│   └── game.js         ← Kod pełnej gry
├── img/
│   └── Partisan_Bushel.png
└── fonts/
    └── codropsicons/   ← Ikony nawigacji
```

---

## 3. Czym jest Three.js?

**Three.js** to biblioteka JavaScript, która umożliwia renderowanie grafiki 3D w przeglądarce za pomocą WebGL. Zamiast pisać niskopoziomowy kod shader'ów, Three.js daje nam wygodne abstrakcje:

### Kluczowe pojęcia Three.js użyte w projekcie:

| Pojęcie | Co to jest | Przykład w kodzie |
|---------|-----------|-------------------|
| **Scene** (Scena) | Kontener na wszystkie obiekty 3D | `scene = new THREE.Scene()` |
| **Camera** (Kamera) | "Oko" przez które patrzymy na scenę | `camera = new THREE.PerspectiveCamera(fov, aspect, near, far)` |
| **Renderer** | Rysuje scenę na ekranie (element `<canvas>`) | `renderer = new THREE.WebGLRenderer()` |
| **Mesh** (Siatka) | Obiekt 3D = Geometria + Materiał | `new THREE.Mesh(geometry, material)` |
| **Geometry** (Geometria) | Kształt obiektu (wierzchołki, ściany) | `new THREE.BoxGeometry(80, 50, 50)` |
| **Material** (Materiał) | Wygląd powierzchni (kolor, połysk, przezroczystość) | `new THREE.MeshPhongMaterial({color: 0xff0000})` |
| **Light** (Światło) | Źródło oświetlenia sceny | `new THREE.DirectionalLight(0xffffff, 0.9)` |
| **Object3D** | Pusty kontener do grupowania obiektów | `new THREE.Object3D()` |
| **Fog** (Mgła) | Efekt mgły — dalsze obiekty zanikają | `new THREE.Fog(color, near, far)` |
| **Shadow** (Cień) | Obiekty mogą rzucać i odbierać cienie | `mesh.castShadow = true` |

### Schemat działania Three.js:

```
┌─────────────────────────────────────────────────┐
│                    SCENA (Scene)                 │
│                                                  │
│   ┌──────────┐  ┌──────────┐  ┌──────────────┐  │
│   │  Światła │  │  Obiekty │  │    Kamera     │  │
│   │  (Light) │  │  (Mesh)  │  │   (Camera)    │  │
│   └──────────┘  └──────────┘  └──────────────┘  │
│                                                  │
└─────────────────────┬───────────────────────────┘
                      │
                      ▼
              ┌──────────────┐
              │   Renderer    │
              │  (WebGL)      │
              └──────┬───────┘
                     │
                     ▼
              ┌──────────────┐
              │   <canvas>    │
              │  (na ekranie) │
              └──────────────┘
```

### Pętla renderowania:

Każda klatka animacji jest rysowana w pętli:
```
function loop() {
    // 1. Aktualizuj pozycje obiektów
    // 2. Aktualizuj animacje
    // 3. Narysuj klatkę:
    renderer.render(scene, camera);
    // 4. Wywołaj ponownie dla następnej klatki:
    requestAnimationFrame(loop);
}
```

---

## 4. Krok 1 — Podstawowa scena (main_step1.js)

Plik `main_step1.js` tworzy najprostszą wersję sceny. Oto co się dzieje krok po kroku:

### 4.1. Inicjalizacja (`init`)

```javascript
window.addEventListener('load', init, false);
```

Po załadowaniu strony uruchamia się `init()`, która:
1. Podpina nasłuchiwanie ruchów myszy
2. Tworzy scenę 3D
3. Tworzy światła
4. Tworzy samolot, morze i niebo
5. Uruchamia pętlę animacji

### 4.2. Tworzenie sceny (`createScene`)

```javascript
scene = new THREE.Scene();
camera = new THREE.PerspectiveCamera(60, WIDTH/HEIGHT, 1, 10000);
camera.position.set(0, 100, 200);   // Kamera patrzy z boku
scene.fog = new THREE.Fog(0xf7d9aa, 100, 950);  // Pomarańczowa mgła
renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true });
```

- **PerspectiveCamera** — Kamera perspektywiczna, naśladuje ludzkie oko. Parametry:
  - `60` — pole widzenia (FOV) w stopniach
  - `WIDTH/HEIGHT` — proporcje ekranu
  - `1, 10000` — zakres widzenia (near/far plane)
- **Fog** — Mgła daje uczucie głębi; kolor `0xf7d9aa` to ciepły pomarańcz
- **WebGLRenderer** — `alpha: true` = przezroczyste tło, `antialias: true` = wygładzanie krawędzi

### 4.3. Światła (`createLights`)

```javascript
hemisphereLight = new THREE.HemisphereLight(0xaaaaaa, 0x000000, 0.9);
shadowLight = new THREE.DirectionalLight(0xffffff, 0.9);
shadowLight.position.set(150, 350, 350);
shadowLight.castShadow = true;
```

- **HemisphereLight** — Światło hemisferyczne: kolor nieba (góra) + kolor ziemi (dół). Symuluje naturalne oświetlenie otoczenia.
- **DirectionalLight** — Światło kierunkowe jak słońce. Rzuca cienie na obiekty.

### 4.4. Samolot (`AirPlane`)

W kroku 1 samolot jest uproszczony — **składa się z prostych prostopadłościanów** (BoxGeometry):

| Element | Geometria | Kolor |
|---------|-----------|-------|
| Kabina (cockpit) | Box 60×50×50 | Czerwony |
| Silnik (engine) | Box 20×50×50 | Biały |
| Statecznik (tailPlane) | Box 15×20×5 | Czerwony |
| Skrzydło (sideWing) | Box 40×8×150 | Czerwony |
| Śmigło (propeller) | Box 20×10×10 | Brązowy |
| Łopatka (blade) | Box 1×100×20 | Ciemnobrązowy |

Cały samolot jest **skalowany do 25%** oryginalnego rozmiaru:
```javascript
airplane.mesh.scale.set(.25, .25, .25);
```

### 4.5. Morze (`Sea`)

```javascript
var geom = new THREE.CylinderGeometry(600, 600, 800, 40, 10);
geom.applyMatrix(new THREE.Matrix4().makeRotationX(-Math.PI/2));
```

Morze to **duży cylinder** (promień 600, wysokość 800) obrócony o 90° — tak, że jego okrągła powierzchnia jest zwrócona do kamery i wygląda jak ocean. Materiał jest **półprzezroczysty** z niebieskim kolorem.

### 4.6. Chmury i Niebo (`Cloud`, `Sky`)

Każda chmura to grupa **3-5 losowych sześcianów** (CubeGeometry 20×20×20) o losowych rozmiarach, pozycjach i rotacjach — razem tworzą "puffy" chmurę w stylu low-poly.

**Sky** tworzy **20 chmur** rozmieszczonych w okręgu wokół środka sceny:
```javascript
var stepAngle = Math.PI * 2 / this.nClouds;  // Kąt między chmurami
var h = 750 + Math.random() * 200;            // Odległość od środka
c.mesh.position.y = Math.sin(a) * h;          // Pozycja Y (okrąg)
c.mesh.position.x = Math.cos(a) * h;          // Pozycja X (okrąg)
```

### 4.7. Sterowanie myszą

Pozycja myszy jest normalizowana do zakresu od -1 do 1:
```javascript
var tx = -1 + (event.clientX / WIDTH) * 2;   // -1 (lewo) do 1 (prawo)
var ty = 1 - (event.clientY / HEIGHT) * 2;    // 1 (góra) do -1 (dół)
```

Funkcja `normalize()` mapuje tę wartość na pozycję samolotu:
```javascript
function normalize(v, vmin, vmax, tmin, tmax) {
    // Mapuje wartość v z zakresu [vmin, vmax] na zakres [tmin, tmax]
}
```

### 4.8. Pętla animacji

```javascript
function loop() {
    updatePlane();                    // Aktualizuj pozycję samolotu
    sea.mesh.rotation.z += .005;     // Obracaj morze (fale)
    sky.mesh.rotation.z += .01;      // Obracaj niebo (chmury lecą)
    renderer.render(scene, camera);  // Narysuj klatkę
    requestAnimationFrame(loop);     // Następna klatka
}
```

---

## 5. Krok 2 — Rozbudowana scena (main_step2.js)

Krok 2 rozszerza scenę o nowe elementy i animacje.

### 5.1. Nowe elementy — Pilot (`Pilot`)

Pilot siedzi w kokpicie i składa się z:

| Element | Opis |
|---------|------|
| **Ciało** (body) | Box 15×15×15, brązowy |
| **Twarz** (face) | Box 10×10×10, różowy |
| **Włosy** (hairs) | 12 małych boxów 4×4×4 ułożonych w siatkę 4×3 |
| **Włosy boczne** | Box 12×4×2 po obu stronach |
| **Okulary** (glass) | Dwa boxy 5×5×5 + poprzeczka 11×1×11 |
| **Uszy** (ears) | Dwa boxy 2×3×2 |

**Animacja włosów** — włosy falują za pomocą funkcji `cos()`:
```javascript
h.scale.y = .75 + Math.cos(this.angleHairs + i/3) * .25;
```
Każdy kosmyk włosów pulsuje sinusoidalnie — daje to efekt "trzepotania na wietrze".

### 5.2. Ulepszone morze — animacja fal

W kroku 2 morze nie jest statyczne — każdy wierzchołek cylindra ma przypisaną falę:
```javascript
this.waves.push({
    y: v.y, x: v.x, z: v.z,         // Oryginalna pozycja
    ang: Math.random() * Math.PI * 2, // Losowa faza
    amp: 5 + Math.random() * 15,      // Amplituda (5-20)
    speed: 0.016 + Math.random() * 0.032  // Prędkość
});
```

W każdej klatce wierzchołki się przesuwają:
```javascript
v.x = vprops.x + Math.cos(vprops.ang) * vprops.amp;
v.y = vprops.y + Math.sin(vprops.ang) * vprops.amp;
vprops.ang += vprops.speed;
```

Daje to **organiczny efekt falujących fal** — każdy wierzchołek porusza się po okręgu z losową prędkością i amplitudą.

### 5.3. Ulepszone sterowanie samolotem

Samolot nie przeskakuje od razu do pozycji myszy, ale **płynnie nadąża** (interpolacja):
```javascript
airplane.mesh.position.y += (targetY - airplane.mesh.position.y) * 0.1;
```

Dodatkowo samolot **przechyla się** w zależności od kierunku ruchu:
```javascript
airplane.mesh.rotation.z = (targetY - airplane.mesh.position.y) * 0.0128;  // Przechył przy wznoszeniu/opadaniu
airplane.mesh.rotation.x = (airplane.mesh.position.y - targetY) * 0.0064;  // Pochylenie przód/tył
```

### 5.4. Ulepszona kabina samolotu

W kroku 2 kabina ma **zmodyfikowane wierzchołki** żeby wyglądała aerodynamicznie:
```javascript
geomCockpit.vertices[4].y -= 10;  // Dolne wierzchołki w dół
geomCockpit.vertices[4].z += 20;  // i szersze
geomCockpit.vertices[6].y += 30;  // Górne wierzchołki wyżej
geomCockpit.vertices[6].z += 20;  // i szersze
```

Zamiast prostego boxa, kabina ma kształt **pochylonego trapezoidalnego profilu** — przód jest wyższy, tył niższy.

Dodano też:
- **Szybę (windshield)** — półprzezroczysta (opacity 0.3)
- **Koła podwozia** — dwa główne koła + jedno tylne (skalowane 50%)
- **Zawieszenie** — obrócony box łączący tył z kołem tylnym

### 5.5. Dynamiczne pole widzenia kamery (FOV)

```javascript
camera.fov = normalize(mousePos.x, -1, 1, 40, 80);
camera.updateProjectionMatrix();
```

Gdy mysz przesuwa się w prawo, FOV rośnie (40→80°) — scena "oddala się", dając efekt przyspieszenia. Gdy w lewo — FOV maleje, scena "przybliża się".

---

## 6. Pełna gra (game.js)

Gra rozszerza krok 2 o mechaniki: monety, wrogów, energię, poziomy i system punktacji.

### 6.1. Zmienne gry (`resetGame`)

Obiekt `game` zawiera **wszystkie parametry gry**:

| Grupa | Parametry | Opis |
|-------|-----------|------|
| **Prędkość** | `speed`, `baseSpeed`, `targetBaseSpeed` | Prędkość gry rośnie z czasem i poziomem |
| **Dystans** | `distance`, `ratioSpeedDistance` | Przebyty dystans = prędkość × czas |
| **Energia** | `energy` (0-100), `ratioSpeedEnergy` | Energia spada z prędkością, game over gdy = 0 |
| **Poziomy** | `level`, `distanceForLevelUpdate` | Co 1000 dystansu = nowy poziom |
| **Samolot** | `planeDefaultHeight`, `planeAmpHeight` | Zakres ruchu, czułość, prędkość |
| **Morze** | `seaRadius`, `wavesMinAmp` | Rozmiar morza, parametry fal |
| **Monety** | `coinValue`, `distanceForCoinsSpawn` | Co 100 dystansu pojawiają się nowe monety |
| **Wrogowie** | `ennemyValue`, `distanceForEnnemiesSpawn` | Co 50 dystansu pojawiają się nowi wrogowie |

### 6.2. System monet (`Coin`, `CoinsHolder`)

**Moneta** to `TetrahedronGeometry(5, 0)` — **czworościan** w kolorze turkusowym.

**CoinsHolder** zarządza monetami za pomocą **wzorca Object Pool**:
```javascript
// Zamiast ciągle tworzyć i usuwać obiekty (drogie!),
// używamy puli gotowych obiektów do ponownego użycia:
if (this.coinsPool.length) {
    coin = this.coinsPool.pop();    // Weź z puli
} else {
    coin = new Coin();              // Stwórz nowy tylko gdy pula pusta
}
```

Monety poruszają się po okręgu wokół osi sceny (jak morze i chmury):
```javascript
coin.angle += game.speed * deltaTime * game.coinsSpeed;
coin.mesh.position.y = -game.seaRadius + Math.sin(coin.angle) * coin.distance;
coin.mesh.position.x = Math.cos(coin.angle) * coin.distance;
```

**Detekcja kolizji** — proste sprawdzenie odległości:
```javascript
var diffPos = airplane.mesh.position.clone().sub(coin.mesh.position.clone());
var d = diffPos.length();
if (d < game.coinDistanceTolerance) {
    // Zebrano monetę! Dodaj energię, efekt cząsteczek
}
```

### 6.3. System wrogów (`Ennemy`, `EnnemiesHolder`)

**Wróg** to `TetrahedronGeometry(8, 2)` — **czworościan z 2 poziomami podziału** (więcej trójkątów) w kolorze czerwonym z połyskiem.

Wrogowie pojawiają się na losowej wysokości i poruszają się po okręgu jak monety. Ilość wrogów na spawn = aktualny `game.level`.

Przy kolizji z wrogiem:
1. Pojawia się eksplozja cząsteczek (czerwonych)
2. Samolot odskakuje (collision displacement)
3. Błysk światła (`ambientLight.intensity = 2`)
4. Odejmowana jest energia

### 6.4. System cząsteczek (`Particle`, `ParticlesHolder`)

**Cząsteczka** to mały czworościan (`TetrahedronGeometry(3, 0)`). Przy eksplozji:

```javascript
TweenMax.to(this.mesh.rotation, speed, {x: Math.random()*12, y: Math.random()*12});
TweenMax.to(this.mesh.scale, speed, {x:.1, y:.1, z:.1});
TweenMax.to(this.mesh.position, speed, {x: targetX, y: targetY, ease: Power2.easeOut});
```

Używa **TweenMax (GreenSock)** do płynnych animacji:
- Rotacja — obraca się losowo
- Skala — kurczy się od 1 do 0.1
- Pozycja — leci w losowym kierunku z wytłumieniem (easeOut)

### 6.5. Sterowanie samolotem w grze (`updatePlane`)

Sterowanie jest bardziej zaawansowane niż w krokach 1-2:

```javascript
// Prędkość samolotu zależy od pozycji X myszy
game.planeSpeed = normalize(mousePos.x, -.5, .5, game.planeMinSpeed, game.planeMaxSpeed);

// Pozycja docelowa
var targetY = normalize(mousePos.y, -.75, .75, minHeight, maxHeight);
var targetX = normalize(mousePos.x, -1, 1, -ampWidth*0.7, -ampWidth);

// Efekt kolizji — odrzuca samolot
targetX += game.planeCollisionDisplacementX;
targetY += game.planeCollisionDisplacementY;

// Płynna interpolacja (lerp)
airplane.mesh.position.y += (targetY - airplane.mesh.position.y) * deltaTime * game.planeMoveSensivity;
```

**Kamera podąża za samolotem** w osi Y:
```javascript
camera.position.y += (airplane.mesh.position.y - camera.position.y) * deltaTime * game.cameraSensivity;
```

### 6.6. Pętla gry (`loop`)

Główna pętla gry wykonuje w każdej klatce:

```
1. Oblicz deltaTime (czas od ostatniej klatki)
2. Jeśli gramy:
   ├── Spawnuj monety (co 100 dystansu)
   ├── Zwiększ prędkość (co 100 dystansu)
   ├── Spawnuj wrogów (co 50 dystansu)
   ├── Sprawdź nowy poziom (co 1000 dystansu)
   ├── Aktualizuj samolot (pozycja, rotacja)
   ├── Aktualizuj dystans
   ├── Aktualizuj energię
   └── Oblicz aktualną prędkość gry
3. Jeśli game over:
   ├── Spowalniaj grę
   ├── Samolot spada i obraca się
   └── Gdy samolot poniżej y=-200 → pokaż "Replay"
4. Obracaj śmigło
5. Obracaj morze
6. Wygaszaj efekt błysku
7. Obracaj monety i wrogów
8. Przesuwaj chmury
9. Animuj fale
10. Renderuj klatkę
11. Następna klatka (requestAnimationFrame)
```

### 6.7. Stany gry

```
"playing"        → Gra trwa, gracz steruje
"gameover"       → Samolot spada, gracz nie kontroluje
"waitingReplay"  → Oczekiwanie na kliknięcie "Replay"
```

### 6.8. UI — interfejs użytkownika

HTML (w `index.html`) zawiera elementy UI aktualizowane z JavaScript:
- **Level** (`#levelValue`) — aktualny poziom
- **Distance** (`#distValue`) — przebyty dystans
- **Energy bar** (`#energyBar`) — pasek energii (zmienia kolor: niebieski → czerwony gdy < 50, mruga gdy < 30)
- **Level circle** (SVG) — okrągły postęp do następnego poziomu (`stroke-dashoffset`)
- **Replay message** — pojawia się po game over

---

## 7. Jak Three.js renderuje scenę — podsumowanie

### Hierarchia obiektów w grze:

```
Scene
├── HemisphereLight          ← Światło otoczenia
├── DirectionalLight         ← Światło słoneczne (cienie)
├── AmbientLight             ← Subtelne światło wypełniające
├── AirPlane (Object3D)      ← Samolot
│   ├── Cabin (Mesh)         ← Kabina
│   ├── Engine (Mesh)        ← Silnik
│   ├── TailPlane (Mesh)     ← Statecznik
│   ├── SideWing (Mesh)      ← Skrzydło
│   ├── Windshield (Mesh)    ← Szyba
│   ├── Propeller (Mesh)     ← Śmigło
│   │   ├── Blade1 (Mesh)
│   │   └── Blade2 (Mesh)
│   ├── WheelProtecR/L (Mesh)← Osłony kół
│   ├── WheelTireR/L/B (Mesh)← Koła
│   ├── Suspension (Mesh)    ← Zawieszenie
│   └── Pilot (Object3D)     ← Pilot
│       ├── Body, Face, Ears
│       ├── Hairs (12 boxów)
│       └── Glasses
├── Sea (Mesh)               ← Morze (cylinder z falami)
├── Sky (Object3D)           ← Niebo
│   └── Cloud × 20           ← Chmury (każda = 3-5 losowych boxów)
├── CoinsHolder (Object3D)   ← Kontener monet
│   └── Coin × n             ← Monety (czworościany)
├── EnnemiesHolder (Object3D)← Kontener wrogów
│   └── Ennemy × n           ← Wrogowie (czworościany)
└── ParticlesHolder (Object3D)← Kontener cząsteczek
    └── Particle × n         ← Cząsteczki eksplozji
```

### Użyte geometrie:

| Geometria | Opis | Gdzie |
|-----------|------|-------|
| `BoxGeometry` | Prostopadłościan | Samolot, pilot, chmury |
| `CylinderGeometry` | Cylinder | Morze |
| `TetrahedronGeometry` | Czworościan | Monety, wrogowie, cząsteczki |
| `CubeGeometry` | Sześcian (alias BoxGeometry) | Klocki chmur |

### Użyte materiały:

| Materiał | Właściwości | Gdzie |
|----------|------------|-------|
| `MeshPhongMaterial` | Kolor + cieniowanie Phong (połysk) | Wszystkie elementy samolotu, morze, chmury |
| `MeshLambertMaterial` | Kolor + cieniowanie Lambert (matowy) | Twarz i włosy pilota |

Oba materiały używają `shading: THREE.FlatShading` — daje to **styl low-poly** (widoczne płaskie ściany zamiast gładkich przejść).

### Technika ruchu — „obracający się świat"

Kluczowa sztuczka: **samolot nie leci do przodu**. Zamiast tego **cały świat obraca się** wokół osi Z:
```javascript
sea.mesh.rotation.z += game.speed * deltaTime;
sky.moveClouds();  // też obraca chmury
```

To daje iluzję lotu, podczas gdy samolot porusza się tylko w górę-dół i lekko w lewo-prawo. Monety i wrogowie również obracają się po okręgu.

---

## 8. Paleta kolorów

| Nazwa | Hex | Kolor | Użycie |
|-------|-----|-------|--------|
| `red` | `#f25346` | 🔴 Czerwony | Kabina, skrzydła, wrogowie |
| `white` | `#d8d0d1` | ⚪ Biały-szary | Silnik, szyba |
| `brown` | `#59332e` | 🟤 Brązowy | Ciało pilota, śmigło |
| `brownDark` | `#23190f` | ⬛ Ciemnobrązowy | Łopatki, koła |
| `pink` | `#F5986E` | 🟠 Różowy-łososiowy | Twarz pilota |
| `yellow` | `#f4ce93` | 🟡 Żółty | (dostępny) |
| `blue` | `#68c3c0` | 🔵 Turkusowy | Morze, monety |

Tło strony to gradient CSS: `#e4e0ba` → `#f7d9aa` (ciepły zachód słońca).
Mgła: `#f7d9aa` (pasuje do tła).

---

## 9. Użyte biblioteki

### Three.js r75
- **Wersja**: Revision 75 (2015)
- **Plik**: `js/three.js` (16 421 linii)
- **Rola**: Silnik 3D — renderowanie WebGL, geometrie, materiały, światła, cienie
- **Strona**: [threejs.org](https://threejs.org)
- **Uwaga**: To stara wersja. Nowsze wersje Three.js (r150+) mają inną strukturę API (np. `BufferGeometry` zamiast `Geometry`, moduły ES6 zamiast globalnych zmiennych).

### GreenSock TweenMax v1.17
- **Wersja**: 1.17.0 (2015)
- **Plik**: `js/TweenMax.js` (2 220 linii)
- **Rola**: Biblioteka animacji tween — płynne przejścia pozycji, skali i rotacji cząsteczek
- **Strona**: [greensock.com](https://greensock.com)
- **Użycie w kodzie**: Tylko w `Particle.prototype.explode()` do animacji eksplozji
- **Uwaga**: Nowsza wersja to GSAP 3 z innym API.

---

> **Autor oryginalnego projektu**: Karim Maaloul (Codrops)
> **Artykuł**: [The Making of "The Aviator": Animating a Basic 3D Scene with Three.js](http://tympanus.net/codrops/?p=26501)
