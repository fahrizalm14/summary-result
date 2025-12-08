# Prompt: Build Reusable Control Panel UI Library (React + Tailwind + lucide-react)

## 📝 Task Description

Berdasarkan styling guide di `agents.md`, buat **library komponen React** untuk Control Panel Dashboard (misalnya untuk NexWage / server management console). Library ini harus:

* Modular dan sangat reusable.
* Menggunakan React best practices.
* Menggunakan Tailwind CSS (sesuai utilitas di `agents.md`).
* Mendukung tampilan **glassmorphism dark dashboard** dengan dukungan light mode.

Semua komponen ditulis dalam **satu file**: `components.jsx`.

---

## 🎨 Design System & Warna

Gunakan tema **dark-modern** dengan aksen biru/ungu + warna status yang jelas.

### 1. Palet Warna Utama

Gunakan kombinasi Tailwind bawaan (bisa kamu map ke brand-mu kalau di `agents.md` sudah ada):

* **Background utama**:

  * Light: `bg-slate-50`
  * Dark: `dark:bg-slate-950`
* **Surface / Card**:

  * Surface utama: `bg-slate-900/70 dark:bg-slate-900/70`
  * Surface nested: `bg-slate-900/40`
  * Glass effect: `backdrop-blur-xl` + `bg-slate-900/60`
* **Primary / Accent**:

  * Primary: `bg-sky-500 hover:bg-sky-400 text-slate-950`
  * Accent: `from-sky-500 to-violet-500` (gradient)
* **Neutral / Border / Text**:

  * Border halus: `border-slate-800/80`
  * Border lebih lembut: `border-slate-800/60`
  * Text utama: `text-slate-50`
  * Text muted: `text-slate-400`

### 2. Warna Status

Status harus **lebih lengkap**, tidak hanya online/offline:

* `online` → hijau: `bg-emerald-500 text-emerald-50 border-emerald-500/40`
* `offline` → abu gelap: `bg-slate-800 text-slate-200 border-slate-700`
* `degraded` → oranye lembut: `bg-amber-500/10 text-amber-300 border-amber-500/40`
* `failed` / `error` → merah: `bg-rose-500/10 text-rose-300 border-rose-500/40`
* `pending` → biru muda: `bg-sky-500/10 text-sky-300 border-sky-500/40`
* `maintenance` → ungu: `bg-purple-500/10 text-purple-300 border-purple-500/40`
* `paused` → netral: `bg-slate-900/80 text-slate-300 border-slate-700`

### 3. Variants & Sizes

**Button variants (minimal 5):**

* `primary` – biru solid (aksi utama)
* `secondary` – abu gelap (aksi sekunder)
* `ghost` – transparan / subtle
* `outline` – border saja, hover jadi berwarna
* `danger` – merah untuk aksi berbahaya
* `icon` – icon-only, bentuk bulat, kecil

**Card variants:**

* `primary` – card utama dengan border & glass effect
* `nested` – card di dalam card lain (lebih ringan)
* `stat` – card untuk metric (gradient + icon besar)

**MetricBar colors (berdasarkan jenis metric):**

* `cpu` → `from-sky-500 to-cyan-400`
* `memory` → `from-violet-500 to-fuchsia-400`
* `disk` → `from-amber-500 to-orange-400`
* `network` → `from-emerald-500 to-teal-400`
* `default` → `from-sky-500 to-violet-500`

**Size options (shared untuk Button / Input / Toggle bila masuk akal):**

* `sm` – compact (badge, toolbar)
* `md` – default (form umum)
* `lg` – besar (CTA, hero action)

---

## 🎯 Requirements

### 1. File & Struktur

Buat file `components.jsx` berisi semua komponen:

```jsx
import React from 'react';
import {
  Cpu,
  Activity,
  Server,
  Power,
  Pause,
  RotateCcw,
  LucideIcon,
} from 'lucide-react';

/** Utility: join class names */
const cn = (...classes) => classes.filter(Boolean).join(' ');

// Mapping variant (lihat di bawah, harus lengkap)
const buttonVariants = { /* ... */ };
const buttonSizes = { /* ... */ };
const statusColors = { /* ... */ };
const cardVariants = { /* ... */ };
const metricBarColors = { /* ... */ };
const tabVariants = { /* ... */ };

// Export all components
export const Button = ({ ...props }) => { /* ... */ };
export const Card = ({ ...props }) => { /* ... */ };
export const StatusBadge = ({ ...props }) => { /* ... */ };
export const Input = ({ ...props }) => { /* ... */ };
export const TextArea = ({ ...props }) => { /* ... */ };
export const Toggle = ({ ...props }) => { /* ... */ };
export const Tab = ({ ...props }) => { /* ... */ };
export const LogEntry = ({ ...props }) => { /* ... */ };

export const Container = ({ ...props }) => { /* ... */ };
export const Grid = ({ ...props }) => { /* ... */ };
export const Section = ({ ...props }) => { /* ... */ };
export const Header = ({ ...props }) => { /* ... */ };

export const StatCard = ({ ...props }) => { /* ... */ };
export const MetricBar = ({ ...props }) => { /* ... */ };
export const ActionButtons = ({ ...props }) => { /* ... */ };
export const FormField = ({ ...props }) => { /* ... */ };
```

---

## 2. Components to Create

### 🔹 Core Components

#### 1. `Button`

* Variants:

  * `primary` | `secondary` | `ghost` | `outline` | `danger` | `icon`
* Sizes:

  * `sm` | `md` (default) | `lg`
* Fitur:

  * Bisa punya **icon di kiri** (`icon` prop).
  * `icon-only` (variant `"icon"`) → width & height sama, content center.
  * Fokus ring: `focus-visible:ring-2 focus-visible:ring-sky-400 focus-visible:ring-offset-2 focus-visible:ring-offset-slate-950`.

**JSDoc:**

```jsx
/**
 * Primary button component
 * @param {'primary'|'secondary'|'ghost'|'outline'|'danger'|'icon'} variant
 * @param {'sm'|'md'|'lg'} size
 * @param {React.ReactNode} children
 * @param {React.ReactNode} icon
 * @param {string} className
 */
export const Button = ({
  variant = 'primary',
  size = 'md',
  icon,
  children,
  className = '',
  ...props
}) => { /* ... */ };
```

**Tailwind base:**

```jsx
const buttonBase =
  'inline-flex items-center justify-center gap-2 rounded-full font-medium transition-all duration-200 ' +
  'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-sky-400 focus-visible:ring-offset-2 focus-visible:ring-offset-slate-950 ' +
  'disabled:opacity-50 disabled:cursor-not-allowed';

const buttonSizes = {
  sm: 'h-8 px-3 text-xs',
  md: 'h-9 px-4 text-sm',
  lg: 'h-11 px-5 text-base',
};

const buttonVariants = {
  primary:
    'bg-sky-500 hover:bg-sky-400 text-slate-950 shadow-lg shadow-sky-500/30',
  secondary:
    'bg-slate-800 hover:bg-slate-700 text-slate-50 border border-slate-700',
  ghost:
    'bg-transparent hover:bg-slate-900/60 text-slate-200 border border-transparent',
  outline:
    'bg-transparent border border-slate-700 hover:border-sky-500 hover:bg-sky-500/5 text-slate-100',
  danger:
    'bg-rose-500/90 hover:bg-rose-400 text-slate-950 shadow-lg shadow-rose-500/30',
  icon:
    'bg-slate-900/60 hover:bg-slate-800/80 text-slate-100 border border-slate-800 h-9 w-9 p-0',
};
```

#### 2. `Card`

* Variants: `primary` | `nested` | `stat`
* Semua card punya:

  * `rounded-2xl` (atau `rounded-xl` untuk nested).
  * `border` + glass (`backdrop-blur-xl`).
  * Transisi halus di hover.

```jsx
const cardVariants = {
  primary:
    'rounded-2xl border border-slate-200/80 bg-slate-50/80 shadow-lg shadow-slate-900/5 backdrop-blur-xl ' +
    'dark:border-slate-800/80 dark:bg-slate-900/70 dark:shadow-sky-500/5',
  nested:
    'rounded-xl border border-slate-200/60 bg-slate-50/70 backdrop-blur-md shadow-sm ' +
    'dark:border-slate-800/60 dark:bg-slate-900/50',
  stat:
    'rounded-2xl border border-slate-800/80 bg-gradient-to-br from-slate-900/90 via-slate-900/50 to-slate-900/90 ' +
    'shadow-lg shadow-sky-500/15 backdrop-blur-xl',
};
```

#### 3. `StatusBadge`

* Input: `status: 'online' | 'offline' | 'degraded' | 'failed' | 'pending' | 'maintenance' | 'paused' | string`
* Auto mapping ke warna.
* Menampilkan dot kecil + label.

```jsx
const statusColors = {
  online:
    'bg-emerald-500/10 text-emerald-300 border border-emerald-500/40',
  offline:
    'bg-slate-900/80 text-slate-300 border border-slate-700',
  degraded:
    'bg-amber-500/10 text-amber-300 border border-amber-500/40',
  failed:
    'bg-rose-500/10 text-rose-300 border border-rose-500/40',
  pending:
    'bg-sky-500/10 text-sky-300 border border-sky-500/40',
  maintenance:
    'bg-purple-500/10 text-purple-300 border border-purple-500/40',
  paused:
    'bg-slate-900/80 text-slate-300 border border-slate-700',
};
```

#### 4. `Input` & `TextArea`

* State:

  * Normal, focus, error.
* Kelas:

  * `bg-slate-900/60 border border-slate-800/80 rounded-xl text-sm text-slate-50`
  * Focus: `focus-visible:ring-2 focus-visible:ring-sky-500 focus-visible:ring-offset-2 focus-visible:ring-offset-slate-950`
  * Placeholder: `placeholder:text-slate-500`

#### 5. `Toggle`

* Switch sederhana:

  * Track: `w-10 h-6 rounded-full bg-slate-800 transition-colors`
  * Thumb: `w-5 h-5 rounded-full bg-slate-100 translate-x-0` (off) → `translate-x-4` (on)
  * Mendukung: `checked`, `onChange`, `disabled`.

#### 6. `Tab`

* Props: `active`, `icon`, `badge`, dll.
* Active:

  * `bg-slate-900/80 text-sky-100 border border-sky-500/60 shadow-md shadow-sky-500/20`
* Inactive:

  * `text-slate-400 hover:text-slate-100 hover:bg-slate-900/40 border border-transparent`

#### 7. `LogEntry`

* Digunakan di activity / message logs.
* Props:

  * `icon`, `title`, `description`, `timestamp`, `status`.
* Layout:

  * `flex items-start gap-3`
  * Icon bulat dengan background halus sesuai status.

---

### 🔹 Layout Components

#### 8. `Container`

* Max width wrapper.
* Kelas:

  * `w-full mx-auto max-w-6xl px-4 lg:px-6`

#### 9. `Grid`

* Props:

  * `cols`: `1 | 2 | 3 | 4`
* Mapping:

  * `cols={2}` → `grid-cols-1 md:grid-cols-2`
  * `cols={4}` → `grid-cols-1 sm:grid-cols-2 lg:grid-cols-4`
* Base:

  * `grid gap-4 md:gap-6`

#### 10. `Section`

* Wrapper dengan margin & spacing konsisten:

  * `space-y-4 md:space-y-6`
  * Bisa pakai `<section>` semantic.

#### 11. `Header`

* Untuk page/section header.
* Props:

  * `title`, `subtitle`, `icon`, `action`.
* Layout:

  * `flex items-center justify-between gap-3`
  * Title group: `flex items-center gap-3`

---

### 🔹 Specialized Components

#### 12. `StatCard`

* Untuk metric (CPU, RAM, dll).
* Props:

  * `icon` (LucideIcon), `label`, `value`, `trend`, `hint`, `color`.
* Design:

  * Pakai `Card` variant `"stat"`.
  * Icon di lingkaran gradien:

    * `bg-gradient-to-br from-sky-500/20 via-sky-500/0 to-sky-500/5`

#### 13. `MetricBar`

* Progress bar:

  * Outer: `h-2.5 rounded-full bg-slate-800/80 overflow-hidden`
  * Inner: `h-full bg-gradient-to-r from-sky-500 to-violet-500`
* Props:

  * `value` (0–100), `colorKey` → pakai `metricBarColors`.

```jsx
const metricBarColors = {
  cpu: 'from-sky-500 to-cyan-400',
  memory: 'from-violet-500 to-fuchsia-400',
  disk: 'from-amber-500 to-orange-400',
  network: 'from-emerald-500 to-teal-400',
  default: 'from-sky-500 to-violet-500',
};
```

#### 14. `ActionButtons`

* Group: Start / Stop / Restart.
* Props:

  * `onStart`, `onStop`, `onRestart`, `size`, `compact`.
* Default:

  * Start: `variant="primary"`
  * Stop: `variant="danger"`
  * Restart: `variant="outline"` + icon `RotateCcw`.

#### 15. `FormField`

* Wrapper label + input.
* Props:

  * `label`, `description`, `error`, `required`, `children`.
* Layout:

  * Wrapper: `space-y-1.5`
  * Label: `text-sm font-medium text-slate-200`
  * Desc: `text-xs text-slate-400`
  * Error: `text-xs text-rose-400`

---

## 3. Color & Variant Mapping (Kode Siap Pakai)

Tambahkan mapping berikut di atas definisi komponen di `components.jsx`:

```jsx
const cn = (...classes) => classes.filter(Boolean).join(' ');

const buttonBase =
  'inline-flex items-center justify-center gap-2 rounded-full font-medium transition-all duration-200 ' +
  'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-sky-400 focus-visible:ring-offset-2 focus-visible:ring-offset-slate-950 ' +
  'disabled:opacity-50 disabled:cursor-not-allowed';

const buttonSizes = {
  sm: 'h-8 px-3 text-xs',
  md: 'h-9 px-4 text-sm',
  lg: 'h-11 px-5 text-base',
};

const buttonVariants = {
  primary:
    'bg-sky-500 hover:bg-sky-400 text-slate-950 shadow-lg shadow-sky-500/30',
  secondary:
    'bg-slate-800 hover:bg-slate-700 text-slate-50 border border-slate-700',
  ghost:
    'bg-transparent hover:bg-slate-900/60 text-slate-200 border border-transparent',
  outline:
    'bg-transparent border border-slate-700 hover:border-sky-500 hover:bg-sky-500/5 text-slate-100',
  danger:
    'bg-rose-500/90 hover:bg-rose-400 text-slate-950 shadow-lg shadow-rose-500/30',
  icon:
    'bg-slate-900/60 hover:bg-slate-800/80 text-slate-100 border border-slate-800 h-9 w-9 p-0',
};

const statusColors = {
  online:
    'bg-emerald-500/10 text-emerald-300 border border-emerald-500/40',
  offline:
    'bg-slate-900/80 text-slate-300 border border-slate-700',
  degraded:
    'bg-amber-500/10 text-amber-300 border border-amber-500/40',
  failed:
    'bg-rose-500/10 text-rose-300 border border-rose-500/40',
  pending:
    'bg-sky-500/10 text-sky-300 border border-sky-500/40',
  maintenance:
    'bg-purple-500/10 text-purple-300 border border-purple-500/40',
  paused:
    'bg-slate-900/80 text-slate-300 border border-slate-700',
};

const cardVariants = {
  primary:
    'rounded-2xl border border-slate-200/80 bg-slate-50/80 shadow-lg shadow-slate-900/5 backdrop-blur-xl ' +
    'dark:border-slate-800/80 dark:bg-slate-900/70 dark:shadow-sky-500/5',
  nested:
    'rounded-xl border border-slate-200/60 bg-slate-50/70 backdrop-blur-md shadow-sm ' +
    'dark:border-slate-800/60 dark:bg-slate-900/50',
  stat:
    'rounded-2xl border border-slate-800/80 bg-gradient-to-br from-slate-900/90 via-slate-900/50 to-slate-900/90 ' +
    'shadow-lg shadow-sky-500/15 backdrop-blur-xl',
};

const metricBarColors = {
  cpu: 'from-sky-500 to-cyan-400',
  memory: 'from-violet-500 to-fuchsia-400',
  disk: 'from-amber-500 to-orange-400',
  network: 'from-emerald-500 to-teal-400',
  default: 'from-sky-500 to-violet-500',
};

const tabVariants = {
  base:
    'inline-flex items-center gap-2 rounded-full px-3 py-1.5 text-xs font-medium transition-all',
  active:
    'bg-slate-900/80 text-sky-100 border border-sky-500/60 shadow-md shadow-sky-500/20',
  inactive:
    'text-slate-400 hover:text-slate-100 hover:bg-slate-900/40 border border-transparent',
};
```

---

## 4. Component List (Ringkas)

* **Button** – Semua aksi utama/sekunder, dengan variasi ukuran & icon.

* **Card** – Wrapper dasar, nested, dan stat card.

* **StatusBadge** – Badge status dengan warna otomatis.

* **Input / TextArea** – Komponen form dengan focus state dan error state.

* **Toggle** – Switch on/off untuk feature flag / enable server.

* **Tab** – Tombol tab dengan active/inactive style.

* **LogEntry** – Baris log aktivitas dengan icon & timestamp.

* **Container** – Wrapper max-width layout.

* **Grid** – Layout grid 2–4 kolom.

* **Section** – Wrapper section dengan spacing konsisten.

* **Header** – Header halaman/section dengan title, subtitle, dan actions.

* **StatCard** – Komponen metric (CPU, RAM, dll).

* **MetricBar** – Progress bar persentase.

* **ActionButtons** – Group start/stop/restart.

* **FormField** – Label + description + error + input.

---

## 5. Usage Examples

### Example 1 – Servers Overview

```jsx
import {
  Button,
  Card,
  StatusBadge,
  StatCard,
  Grid,
  Header,
  ActionButtons,
} from './components';

export const ServersSection = () => (
  <Section>
    <Card variant="primary">
      <Header
        title="Servers"
        subtitle="Monitor status dan performa semua instance"
        action={<Button variant="primary">+ Add Server</Button>}
      />

      <Card variant="nested" className="mt-4 space-y-4">
        <div className="flex items-center justify-between gap-3">
          <div className="flex items-center gap-3">
            <StatusBadge status="online" />
            <div>
              <div className="font-medium text-slate-50">
                Production Server
              </div>
              <div className="text-xs text-slate-400">
                us-east-1 · 4 vCPU · 16 GB RAM
              </div>
            </div>
          </div>

          <ActionButtons
            onStart={() => {}}
            onStop={() => {}}
            onRestart={() => {}}
          />
        </div>

        <Grid cols={4} className="mt-2">
          <StatCard icon={Cpu} label="CPU" value="45%" color="cpu" />
          <StatCard icon={Activity} label="RAM" value="62%" color="memory" />
        </Grid>
      </Card>
    </Card>
  </Section>
);
```

### Example 2 – Settings Form

```jsx
import { Card, FormField, Input, Toggle, Button } from './components';

export const SettingsForm = () => (
  <Card variant="primary" className="space-y-4">
    <Header
      title="Auto Scaling"
      subtitle="Konfigurasi skala otomatis berdasarkan beban server"
    />

    <FormField
      label="Auto scale"
      description="Aktifkan untuk menambah instance ketika beban tinggi."
    >
      <Toggle checked={true} onChange={() => {}} />
    </FormField>

    <FormField
      label="Max CPU (%)"
      description="Batas CPU sebelum auto scale dinaikkan."
      error=""
    >
      <Input type="number" defaultValue={80} />
    </FormField>

    <div className="flex justify-end gap-2 pt-2">
      <Button variant="ghost">Cancel</Button>
      <Button variant="primary">Save changes</Button>
    </div>
  </Card>
);
```

---
