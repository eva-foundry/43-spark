# Skill: Internationalisation — EN / FR

> Load this skill when adding, editing, or reviewing any user-visible text in EVA-JP.  
> Standard: Government of Canada Official Languages Act (English and French).

---

## The Golden Rule

**Every character the user can read must come from `t("key")`.**  
No exceptions — not button labels, not `aria-label`s, not alt text, not error messages, not aria-describedby strings.

---

## Setup

```tsx
// src/i18n/i18n.tsx
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';
import enResources from './locales/en/resources_en.json';
import frResources from './locales/fr/resources_fr.json';

i18n
  .use(LanguageDetector)      // detects navigator.language on first visit
  .use(initReactI18next)
  .init({
    fallbackLng: 'en',
    supportedLngs: ['en', 'fr'],
    resources: {
      en: { translation: enResources },
      fr: { translation: frResources },
    },
    interpolation: { escapeValue: false },
    detection: {
      order: ['navigator', 'localStorage'],
      caches: ['localStorage'],
    },
  });

// Update html lang attribute on change
i18n.on('languageChanged', (lng) => {
  document.documentElement.lang = lng;   // AC-05
});

export default i18n;
```

---

## Key Convention

Keys are the human-readable **English** string exactly as it appears in the UI.

```json
// ✅ resources_en.json
{
  "Send question": "Send question",
  "Clear chat": "Clear chat",
  "Switch interface language to French": "Switch interface language to French",
  "Loading the folders, Please wait": "Loading the folders, please wait",
  "Translation successful": "Translation successful"
}

// ✅ resources_fr.json
{
  "Send question": "Envoyer la question",
  "Clear chat": "Effacer le clavardage",
  "Switch interface language to French": "Passer l'interface en français",
  "Loading the folders, Please wait": "Chargement des dossiers, veuillez patienter",
  "Translation successful": "Traduction réussie"
}

// ❌ Never
{
  "sendQuestion": "Send question"   // camelCase key — wrong convention
}
```

---

## Usage in Components

```tsx
import { useTranslation } from 'react-i18next';

const MyComponent: React.FC = () => {
  const { t } = useTranslation();

  return (
    <Button
      aria-label={t("Send question")}  // ✅ aria-label via t()
      icon={<Send28Filled aria-hidden="true" />}
      onClick={handleSend}
    >
      {t("Send question")}              {/* ✅ visible label via t() */}
    </Button>
  );
};
```

---

## Language Toggle (Header)

```tsx
import { useTranslation } from 'react-i18next';

const LanguageToggle: React.FC = () => {
  const { i18n, t } = useTranslation();
  const isEn = i18n.language.startsWith('en');

  const toggle = () => {
    const next = isEn ? 'fr' : 'en';
    i18n.changeLanguage(next);   // triggers languageChanged → sets document.lang
  };

  return (
    <Button
      appearance="subtle"
      onClick={toggle}
      aria-label={isEn
        ? t("Switch interface language to French")
        : t("Switch interface language to English")}
    >
      {isEn ? t("Language") + ': FR' : t("Language") + ': EN'}
    </Button>
  );
};
```

---

## Dates and Numbers

```tsx
// Always use the active locale — never hardcode format
const { i18n } = useTranslation();

// Date
const formattedDate = new Intl.DateTimeFormat(i18n.language, {
  year: 'numeric', month: 'long', day: 'numeric',
}).format(new Date(file.uploadDate));

// Number
const formattedCount = new Intl.NumberFormat(i18n.language).format(resultCount);
```

---

## Plurals

```json
// resources_en.json
{
  "result_one": "{{count}} result",
  "result_other": "{{count}} results"
}

// resources_fr.json  (French: 0 and 1 are singular)
{
  "result_one": "{{count}} résultat",
  "result_other": "{{count}} résultats"
}
```

```tsx
// Usage
t('result', { count: resultCount })
```

---

## Locale File Maintenance

When adding a new string to the UI:
1. Add the key/value to `resources_en.json`
2. Add the **French translation** (not a copy of English) to `resources_fr.json`
3. Use the exact same key in both files

**FR translation rules**:
- Must be actual French, not Google-translated English
- Formal register (`vous`, not `tu`)
- Terminology follows GC translation bureau guidelines where applicable
- Never leave the value identical to the English key — this breaks AC-03 verification

---

## Complete i18n Key List for EVA-JP

### Layout Shell

```json
"Chat": "Clavardage",
"Manage Content": "Gérer le contenu",
"Translator": "Traducteur",
"URL Scrapper": "Extracteur d'URL",
"Math Assistant": "Assistant mathématique",
"Tabular Data Assistant": "Assistant de données tabulaires",
"Welcome": "Bienvenue",
"Loading...If more than 15 seconds, please refresh your browser": "Chargement... Si cela prend plus de 15 secondes, veuillez actualiser votre navigateur",
"Switch interface language to English": "Passer l'interface en anglais",
"Switch interface language to French": "Passer l'interface en français",
"Language": "Langue",
"Primary navigation": "Navigation principale",
"Open menu": "Ouvrir le menu",
"Close menu": "Fermer le menu",
"Select user group": "Sélectionner le groupe d'utilisateurs",
"Change Group": "Changer de groupe",
"Preview": "Aperçu (fonctionnalité)",
"Skip to main content": "Passer au contenu principal"
```

### Chat Screen

```json
"Send question": "Envoyer la question",
"Clear chat": "Effacer le clavardage",
"Adjust settings": "Ajuster les paramètres",
"Chat messages": "Messages de clavardage",
"Work documents only": "Documents de travail seulement",
"Web search": "Recherche web",
"Work documents and web search": "Documents de travail et recherche web",
"New chat": "Nouveau clavardage",
"Loading answer": "Chargement de la réponse",
"Regenerate answer": "Régénérer la réponse",
"Show analysis": "Afficher l'analyse",
"Concise": "Concis",
"Balanced": "Équilibré",
"Detailed": "Détaillé",
"Precise": "Précis",
"Creative": "Créatif",
"Give feedback": "Donner de la rétroaction"
```

### Manage Content / Upload

```json
"Upload Files": "Téléverser des fichiers",
"Upload Status": "État du téléversement",
"URL Scraper": "Extracteur d'URL",
"Supported file types": "Types de fichiers pris en charge"
```

### Translator

```json
"Select folder": "Sélectionner un dossier",
"Select file": "Sélectionner un fichier",
"Target language": "Langue cible",
"English": "Anglais",
"French": "Français",
"Translate": "Traduire",
"Loading the folders, Please wait": "Chargement des dossiers, veuillez patienter",
"Folders loaded successfully!": "Dossiers chargés avec succès!",
"Translation successful": "Traduction réussie",
"Translation failed": "Traduction échouée",
"Download translated file": "Télécharger le fichier traduit"
```

### URL Scraper

```json
"URLs (one per line)": "URL (une par ligne)",
"Blacklisted URLs (one per line)": "URL sur liste noire (une par ligne)",
"Max links": "Nombre maximal de liens",
"Preview links": "Aperçu des liens",
"Check all": "Tout cocher",
"Uncheck all": "Tout décocher",
"Cancel": "Annuler",
"Selected: {{count}} links": "Sélectionné : {{count}} lien(s)"
```

### TDA Screen

```json
"Tabular Data Assistant": "Assistant de données tabulaires",
"Upload a CSV file": "Téléverser un fichier CSV",
"Drop CSV here or click to select": "Déposez un CSV ici ou cliquez pour sélectionner",
"Enter your question": "Entrez votre question",
"Analyse": "Analyser",
"Analysis result": "Résultat de l'analyse",
"Generated chart": "Graphique généré",
"An error occurred.": "Une erreur est survenue."
```

### Math Tutor

```json
"Math Tutor": "Tuteur de maths",
"Math problem": "Problème mathématique",
"Give Me Clues": "Donnez-moi des indices",
"Show Me How to Solve It": "Montrez-moi comment le résoudre",
"Show Me the Answer": "Montrez-moi la réponse",
"Math answer": "Réponse mathématique",
"Max retries reached. Last error:": "Nombre maximal de tentatives atteint. Dernière erreur :"
```

### 404

```json
"Page not found": "Page introuvable",
"The page you are looking for does not exist.": "La page que vous cherchez n'existe pas.",
"Return to Chat": "Retour au clavardage"
```
