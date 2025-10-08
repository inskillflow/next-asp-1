# Phase 2 — Intégration des pages web (Next.js App Router)

<ul>
  <li><a href="#sec-phase2">Phase 2 — Intégration des pages web</a></li>
  <li><a href="#sec-intro2">1 – Introduction & livrables</a></li>
  <li><a href="#sec-archi2">2 – Arborescence & routing (App Router)</a></li>
  <li>
    <a href="#sec-pages2">3 – Pages à livrer (exhaustif, liens & I/O)</a>
    <ul>
      <li><a href="#sec-courses-pages">3.1 Courses</a> — <a href="#p-courses-list">liste</a> · <a href="#p-courses-create">create</a> · <a href="#p-courses-detail">detail</a> · <a href="#p-courses-edit">edit</a></li>
      <li><a href="#sec-instructors-pages">3.2 Instructors</a> — <a href="#p-instructors-list">liste</a> · <a href="#p-instructors-create">create</a> · <a href="#p-instructors-detail">detail</a> · <a href="#p-instructors-edit">edit</a></li>
      <li><a href="#sec-students-pages">3.3 Students</a> — <a href="#p-students-list">liste</a> · <a href="#p-students-create">create</a> · <a href="#p-students-detail">detail</a> · <a href="#p-students-edit">edit</a></li>
      <li><a href="#sec-enrollments-pages">3.4 Enrollments</a> — <a href="#p-enrollments-list">liste</a> · <a href="#p-enrollments-create">create</a> · <a href="#p-enrollments-detail">detail</a> · <a href="#p-enrollments-edit">edit</a></li>
      <li><a href="#sec-health-page">3.5 Health</a> — <a href="#p-health">/health</a></li>
    </ul>
  </li>
  <li><a href="#sec-ui2">4 – Conventions UI (navigation, états, accessibilité)</a></li>
  <li><a href="#sec-fetch2">5 – Data fetching (GET/POST/PUT/DELETE)</a></li>
  <li><a href="#sec-forms2">6 – Formulaires & validation côté client</a></li>
  <li><a href="#sec-files2">7 – Fichiers exigés : <code>layout.tsx</code>, <code>page.module.css</code>, <code>page.tsx</code></a></li>
  <li><a href="#sec-compat2">8 – Compatibilité Phase 3 (auth prête)</a></li>
  <li><a href="#sec-check2">9 – Checklist de rendu</a></li>
</ul>

<br/><br/>

<h1 id="sec-phase2">Phase 2 — Intégration des pages web</h1>

<h2 id="sec-intro2">1 – Introduction & livrables</h2>
Objectif : **intégrer des pages Next.js** correspondant **1:1** aux endpoints de la Phase 1 pour lister, créer, consulter, modifier et supprimer chaque ressource.  
**Livrables obligatoires** (mention de l’énoncé) :  
- Intégrer les pages web correspondant aux endpoints API.  
- Inclure dans le projet : **`layout.tsx`**, **`page.module.css`**, **`page.tsx`** (racine).  
**Contraintes** : pas d’auth à cette étape (tout public). Préparer la navigation et des hooks qui **supporteront** l’auth/roles en Phase 3 **sans changer les URLs**.

<h2 id="sec-archi2">2 – Arborescence & routing (App Router)</h2>

```
app/
├─ layout.tsx                # Layout global (header/nav/footer)
├─ page.tsx                  # Accueil: liens vers les sections
├─ page.module.css           # Styles de base
├─ health/
│  └─ page.tsx               # /health
├─ courses/
│  ├─ page.tsx               # /courses (liste)
│  ├─ create/
│  │  └─ page.tsx            # /courses/create
│  └─ [id]/
│     ├─ page.tsx            # /courses/{id} (détail)
│     └─ edit/
│        └─ page.tsx         # /courses/{id}/edit
├─ instructors/
│  ├─ page.tsx
│  ├─ create/page.tsx
│  └─ [id]/{page.tsx,edit/page.tsx}
├─ students/
│  ├─ page.tsx
│  ├─ create/page.tsx
│  └─ [id]/{page.tsx,edit/page.tsx}
└─ enrollments/
   ├─ page.tsx
   ├─ create/page.tsx
   └─ [id]/{page.tsx,edit/page.tsx}
```

> Chaque page **consomme les endpoints REST** de la Phase 1. En Phase 3, on ajoute les guards (401/403) sans changer ces routes.

<br/>

<h2 id="sec-pages2">3 – Pages à livrer (exhaustif, liens & I/O)</h2>

> Notation : **GET** = lecture ; **POST** = création ; **PUT** = maj ; **DELETE** = suppression. Les **actions** déclenchent les appels API correspondants.

<h3 id="sec-courses-pages">3.1 Courses</h3>

<h4 id="p-courses-list">/courses — Liste</h4>
- **Affiche** la table paginée des cours (title, level, price, instructorId, createdAt).  
- **Actions** : Lien **Détail**, **Éditer**, **Supprimer**, bouton **Créer**.  
- **API** : `GET /api/courses?page=&size=&sort=&level=&instructorId=&priceMin=&priceMax`  
- **Erreurs** : afficher un bandeau si `500`.

<h4 id="p-courses-create">/courses/create — Créer</h4>
- **Form** → `POST /api/courses` (title, description, instructorId, duration, maxStudents, price, level).  
- **Validation client** (mêmes règles que Phase 1).  
- **Succès** : redirection vers **/courses** + toast « Course created ».  
- **Erreurs** : `422/409` (montrer messages), `500`.

<h4 id="p-courses-detail">/courses/{id} — Détail</h4>
- **API** : `GET /api/courses/{id}`.  
- **Affiche** tous les champs.  
- **Actions** : **Éditer**, **Supprimer**, **Retour**.

<h4 id="p-courses-edit">/courses/{id}/edit — Modifier</h4>
- **Form** préremplie → `PUT /api/courses/{id}`.  
- **Succès** : retour page détail.  
- **Erreurs** : `422/404/409/500`.

<hr/>

<h3 id="sec-instructors-pages">3.2 Instructors</h3>

<h4 id="p-instructors-list">/instructors — Liste</h4>
- **API** : `GET /api/instructors?page=&size=&sort=&email=&specialty`.  
- **Actions** : Détail / Éditer / Supprimer / Créer.

<h4 id="p-instructors-create">/instructors/create — Créer</h4>
- **Form** → `POST /api/instructors`.  
- Gérer `422/409`.

<h4 id="p-instructors-detail">/instructors/{id} — Détail</h4>
- **API** : `GET /api/instructors/{id}`.

<h4 id="p-instructors-edit">/instructors/{id}/edit — Modifier</h4>
- **Form** → `PUT /api/instructors/{id}`.

<hr/>

<h3 id="sec-students-pages">3.3 Students</h3>

<h4 id="p-students-list">/students — Liste</h4>
- **API** : `GET /api/students?page=&size=&sort=&email=&level`.

<h4 id="p-students-create">/students/create — Créer</h4>
- **Form** → `POST /api/students`.

<h4 id="p-students-detail">/students/{id} — Détail</h4>
- **API** : `GET /api/students/{id}`.

<h4 id="p-students-edit">/students/{id}/edit — Modifier</h4>
- **Form** → `PUT /api/students/{id}`.

<hr/>

<h3 id="sec-enrollments-pages">3.4 Enrollments</h3>

<h4 id="p-enrollments-list">/enrollments — Liste</h4>
- **API** : `GET /api/enrollments?page=&size=&sort=&status=&courseId=&studentId`.  
- Afficher `status`, `grade`, `courseId`, `studentId`.

<h4 id="p-enrollments-create">/enrollments/create — Créer</h4>
- **Form** → `POST /api/enrollments` (courseId, studentId, status, grade?).  
- **Tip** : autosuggest course/student (petit select alimenté par `/api/courses?size=50`, `/api/students?size=50`).

<h4 id="p-enrollments-detail">/enrollments/{id} — Détail</h4>
- **API** : `GET /api/enrollments/{id}`.

<h4 id="p-enrollments-edit">/enrollments/{id}/edit — Modifier</h4>
- **Form** → `PUT /api/enrollments/{id}` (status/grade).

<hr/>

<h3 id="sec-health-page">3.5 Health</h3>

<h4 id="p-health">/health — Statut</h4>
- **API** : `GET /api/health`  
- Afficher badge **ok** / **degraded** + horodatage.

<br/>

<h2 id="sec-ui2">4 – Conventions UI (navigation, états, accessibilité)</h2>

* **Navigation globale** (header) : liens **Courses / Instructors / Students / Enrollments / Health**.
* **Actions cohérentes** : bouton « Create », menu contextuel « Edit / Delete / View ».
* **États** : loading (squelettes), empty-state, error-state.
* **A11y** : labels de formulaire, `aria-live="polite"` pour toasts, focus management après submit.
* **Tables** : pagination simple (page/size), tri minimal sur `createdAt` (optionnel).

<h2 id="sec-fetch2">5 – Data fetching (GET/POST/PUT/DELETE)</h2>

* **GET** côté serveur (Server Components) quand c’est statique (liste/détail), via `await fetch(url, { cache: 'no-store' })`.
* **Actions mutantes** en **Form Actions** ou handlers côté client (via `use client`) :

  * `POST` create
  * `PUT` edit
  * `DELETE` remove
* **Gestion erreurs** : lire le JSON d’erreur (format Phase 1) et afficher `details.fieldErrors` s’il y en a.

Exemple utilitaire `lib/http.ts`:

```ts
export async function api<T>(input: string, init?: RequestInit): Promise<T> {
  const res = await fetch(input, { ...init, headers: { 'Content-Type': 'application/json', ...(init?.headers||{}) } })
  if (!res.ok) {
    const err = await res.json().catch(() => ({}))
    throw Object.assign(new Error(err?.message || 'Request failed'), { status: res.status, details: err?.details })
  }
  return res.status === 204 ? (undefined as T) : await res.json()
}
```

<h2 id="sec-forms2">6 – Formulaires & validation côté client</h2>

* **Validation**: Zod (même schémas que Phase 1) – optionnel mais recommandé pour UX.
* **Affichage erreurs** : sous chaque champ + résumé en haut (422).
* **Navigation**: après succès, `redirect()` vers la liste/détail + toast.

<h2 id="sec-files2">7 – Fichiers exigés : <code>layout.tsx</code>, <code>page.module.css</code>, <code>page.tsx</code></h2>

**`app/layout.tsx` (exemple minimal)**

```tsx
// app/layout.tsx
import './page.module.css'
import Link from 'next/link'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="fr">
      <body>
        <header className="nav">
          <nav>
            <Link href="/">EduTrack</Link>
            <ul>
              <li><Link href="/courses">Courses</Link></li>
              <li><Link href="/instructors">Instructors</Link></li>
              <li><Link href="/students">Students</Link></li>
              <li><Link href="/enrollments">Enrollments</Link></li>
              <li><Link href="/health">Health</Link></li>
            </ul>
          </nav>
        </header>
        <main className="container">{children}</main>
        <footer className="footer">© EduTrack</footer>
      </body>
    </html>
  )
}
```

**`app/page.module.css` (exemple sobre)**

```css
:root { color-scheme: light dark; }
body { margin:0; font-family: system-ui, sans-serif; }
.container { max-width: 1080px; margin: 2rem auto; padding: 0 1rem; }
.nav nav { display:flex; gap:1rem; align-items:center; padding: 0.75rem 1rem; border-bottom: 1px solid #e5e7eb; }
.nav ul { display:flex; gap:0.75rem; list-style:none; margin:0; padding:0; }
.nav a { text-decoration:none; }
.table { width:100%; border-collapse: collapse; }
.table th, .table td { padding: 0.5rem 0.75rem; border-bottom: 1px solid #e5e7eb; }
.actions { display:flex; gap:0.5rem; }
.form { display:grid; gap:0.75rem; max-width: 640px; }
.input { padding:0.5rem 0.75rem; border:1px solid #d1d5db; border-radius:8px; }
.button { padding:0.5rem 0.85rem; border:1px solid #d1d5db; border-radius:8px; background:#f9fafb; cursor:pointer; }
.footer { padding: 1rem; color:#6b7280; text-align:center; }
.error { color: #b91c1c; }
.success { color: #047857; }
```

**`app/page.tsx` (accueil avec liens)**

```tsx
// app/page.tsx
import Link from 'next/link'

export default function Home() {
  return (
    <section>
      <h1>EduTrack — Phase 2</h1>
      <p>Sélectionne une ressource :</p>
      <ul>
        <li><Link href="/courses">Courses</Link></li>
        <li><Link href="/instructors">Instructors</Link></li>
        <li><Link href="/students">Students</Link></li>
        <li><Link href="/enrollments">Enrollments</Link></li>
        <li><Link href="/health">Health</Link></li>
      </ul>
    </section>
  )
}
```

> Tu peux décliner ces patterns pour chaque page listée en 3.x (liste/create/detail/edit). Les styles peuvent rester simples (CSS Module unique).

<h2 id="sec-compat2">8 – Compatibilité Phase 3 (auth prête)</h2>

* Les **routes de pages ne changent pas**.
* Prévois dans `lib/http.ts` un header `Authorization` **optionnel** (désactivé en Phase 2). En Phase 3, tu le rempliras automatiquement (JWT/Clerk/NextAuth).
* En cas de `401/403` (Phase 3), affiche une page **Unauthorized/Forbidden** réutilisable.

<h2 id="sec-check2">9 – Checklist de rendu</h2>

* [ ] `app/layout.tsx`, `app/page.tsx`, `app/page.module.css` présents.
* [ ] Pages **liste/create/detail/edit** pour **courses, instructors, students, enrollments**.
* [ ] Page **/health**.
* [ ] Appels API branchés sur **les endpoints Phase 1** ; messages d’erreur affichés (`422/409/404/500`).
* [ ] Navigation globale fonctionnelle.
* [ ] Validation formulaire côté client (recommandée) et redirections après succès.

