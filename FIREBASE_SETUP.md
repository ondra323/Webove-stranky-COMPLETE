# 🔥 Firebase — Návod pro vývojáře

## Krok 1: Vytvoření projektu

1. Jděte na https://console.firebase.google.com
2. Klikněte **"Add project"** → pojmenujte ho (např. `kadlec-web`)
3. Google Analytics: můžete přeskočit

---

## Krok 2: Přidání Web App

1. V projektu klikněte na ikonu **</>** (Web)
2. Zadejte název (např. `kadlec-blog`)
3. Zkopírujte vygenerovaný `firebaseConfig` objekt
4. Vložte klíče na **dvě místa** v souborech:
   - `blog.html` — sekce `FIREBASE SDK` na začátku
   - `admin.html` — sekce `FIREBASE SDK` na začátku

Hledejte komentáře `// ⚙️ VÝVOJÁŘ: Vložte zde...`

---

## Krok 3: Firestore Database

1. V Firebase Console → **Firestore Database** → **Create database**
2. Zvolte **Production mode** (bezpečnější)
3. Vyberte region: `europe-west3` (Frankfurt)

### Pravidla přístupu (Firestore Rules)

Přejděte na **Firestore → Rules** a vložte:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Veřejně čitelné pouze publikované články
    match /articles/{articleId} {
      allow read: if resource.data.status == 'published';
      allow write: if request.auth != null;
    }

  }
}
```

**Co to znamená:**
- `allow read`: Návštěvníci vidí jen publikované články (status == 'published')
- `allow write`: Zapisovat může pouze přihlášený admin (Firebase Auth)

---

## Krok 4: Firebase Authentication

1. Firebase Console → **Authentication** → **Get started**
2. Záložka **Sign-in method** → povolte **Email/Password**
3. Záložka **Users** → **Add user**
   - Email: `ondrej@vasedomena.cz`
   - Heslo: (zvolte silné heslo)

Tento email a heslo budete zadávat do přihlašovacího formuláře v `admin.html`.

---

## Krok 5: Struktura dat v Firestore

Každý článek je dokument v kolekci `articles` s těmito poli:

| Pole          | Typ        | Popis                              |
|---------------|------------|------------------------------------|
| `title`       | string     | Nadpis článku                      |
| `excerpt`     | string     | Krátký popis (perex)               |
| `content`     | string     | HTML obsah (z Quill editoru)       |
| `category`    | string     | Hypotéky / Investice / Penze / ... |
| `imageUrl`    | string     | URL náhledového obrázku            |
| `status`      | string     | `published` nebo `draft`           |
| `createdAt`   | timestamp  | Datum vytvoření                    |
| `publishedAt` | timestamp  | Datum publikování                  |
| `updatedAt`   | timestamp  | Datum poslední úpravy              |

---

## Krok 6: Firestore Indexes

Pro správné řazení + filtrování musíte vytvořit **composite index**:

1. Firebase Console → **Firestore → Indexes → Composite**
2. Přidejte index:
   - Collection: `articles`
   - Field 1: `status` (Ascending)
   - Field 2: `publishedAt` (Descending)
3. Klikněte **Save**

> Alternativně: při prvním načtení blogu se zobrazí chybová hláška v konzoli s přímým odkazem na vytvoření indexu — stačí na něj kliknout.

---

## Nasazení (Deployment)

### Varianta A: Firebase Hosting (doporučeno, zdarma)
```bash
npm install -g firebase-tools
firebase login
firebase init hosting
firebase deploy
```

### Varianta B: Jakýkoli statický hosting
Stačí nahrát oba soubory (`blog.html`, `admin.html`, `gdpr.html`) na:
- Netlify (netlify.com) — přetáhněte složku, hotovo
- Vercel (vercel.com)
- Vlastní server (FTP)

### Admin URL
Soubor `admin.html` je dostupný na URL jako:
`https://vasadomena.cz/admin.html`

Doporučujeme přejmenovat na méně předvídatelnou URL, např.:
`admin-k9x2m.html`

---

## Bezpečnost — Checklist

- [ ] Firebase klíče vloženy do obou souborů
- [ ] Firestore Rules nastaveny (viz Krok 3)
- [ ] Authentication zapnuta s Email/Password
- [ ] Admin účet vytvořen se silným heslem (12+ znaků)
- [ ] Admin soubor pojmenován neprediktabilně

---

## Podpora

Při problémech hledejte v **Firebase Console → Firestore → Usage** nebo v Chrome DevTools (F12) → Console.
