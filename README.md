<div align="center">

# 🎬 Movies Library

**Εφαρμογή αναζήτησης & ανάδειξης δημοφιλών ταινιών**  
*(English version follows after the Greek one — scroll down)*

</div>

---

## 🇬🇷 Ελληνικά

### 1. Περιγραφή
Το **Movies Library** είναι μία μοντέρνα Single Page Application (SPA) γραμμένη σε React, που επιτρέπει στον χρήστη να αναζητά ταινίες μέσω του TMDB API και ταυτόχρονα να παρακολουθείται η «τάση» των αναζητήσεων. Κάθε όρος αναζήτησης αποθηκεύεται σε βάση (Appwrite) μαζί με την ταινία που εμφανίστηκε πρώτη στα αποτελέσματα. Με βάση το πόσες φορές αναζητήθηκε ένας όρος, παράγεται δυναμικά μια λίστα «Trending Movies» (τα πιο συχνά αναζητημένα). 

Κύρια χαρακτηριστικά:
- 🔍 Ζωντανή (real‑time αισθητική) αναζήτηση ταινιών με debounce (περιορισμός περιττών κλήσεων API).
- ⭐ Προβολή βασικών στοιχείων (αφίσα, βαθμολογία, γλώσσα, έτος).
- 📈 Δυναμικά «Trending Movies» βάσει πραγματικής χρήσης (μετρική count ανά search term).
- 🛡️ Graceful αποτυχίες (μηνύματα σφάλματος, fallback εικόνα `no-movie.png`).
- ⚡ Ταχύτατη ανάπτυξη & HMR μέσω Vite.

### 2. Αρχιτεκτονική
Η αρχιτεκτονική παραμένει ελαφριά, προσανατολισμένη στο UI και στα data flows:

| Στρώμα | Περιγραφή |
| ------ | --------- |
| Presentation (React Components) | `App.jsx` ως root container, επιμέρους components: `Search`, `MovieCard`, `Spinner`. |
| Data Fetching | Απλές `fetch` κλήσεις στο TMDB API με Authorization header (Bearer Token). |
| State Management | Τοπικά React hooks (`useState`, `useEffect`) + `useDebounce` από το πακέτο `react-use` για debounce logic. |
| Persistence Layer | Appwrite Database για αποθήκευση / ενημέρωση όρων αναζήτησης & παραγωγή trending λίστας. |
| Derivative Data | Trending movies παράγονται με ταξινόμηση κατά `count` (descending) μέσω Appwrite `Query.orderDesc`. |

#### Ροή Δεδομένων (High Level)
1. Ο χρήστης πληκτρολογεί στο πεδίο αναζήτησης (`Search` component).  
2. Το `searchTerm` ενημερώνεται άμεσα, αλλά το `debouncedSearchTerm` ενημερώνεται μετά από 300ms αδράνειας.  
3. Το effect στο `App.jsx` καλεί `fetchMovies(debouncedSearchTerm)` (TMDB API).  
4. Αν υπάρχει query & αποτελέσματα, καλείται `updateSearchCount` → Appwrite: είτε δημιουργεί νέο document είτε αυξάνει το `count`.  
5. Στην πρώτη φόρτωση καλείται `getTrendingMovies()` → φέρνει τα Top 5 search-based entries.  
6. Προβάλλονται δύο λίστες: Trending (μεταγλωττισμένα από τη βάση) & All Movies (τρέχον αποτέλεσμα αναζήτησης ή δημοφιλείς).  

#### Error Handling & Fallbacks
- Μη επιτυχές HTTP response → μήνυμα σφάλματος.
- Έλλειψη `poster_path` → χρήση `/no-movie.png`.
- Άδεια ή ελλιπή πεδία → ασφαλείς default τιμές (π.χ. `N/A`).

### 3. Τεχνολογίες & Βιβλιοθήκες
- **React 19**: Δημιουργία UI με components & hooks.
- **Vite**: Development server, HMR, βελτιστοποιημένο build.
- **Tailwind CSS (v4)**: Utility-first styling (μέσω integration @tailwindcss/vite).
- **Appwrite SDK**: Client-side κλήσεις για Database operations (CRUD documents & queries).
- **TMDB API**: Πηγή δεδομένων ταινιών (τίτλος, poster, metadata).
- **react-use (useDebounce)**: Απλοποιεί το debounce logic της αναζήτησης.
- **ESLint**: Linting & συνεπές style.

### 4. Environment Μεταβλητές (.env)
Δημιούργησε ένα αρχείο `.env` (ή `.env.local`) στη ρίζα:
```
VITE_TMDB_API_KEY=<<TMDB_BEARER_TOKEN>>
VITE_APPWRITE_ENDPOINT=https://cloud.appwrite.io/v1
VITE_APPWRITE_PROJECT_ID=<<PROJECT_ID>>
VITE_APPWRITE_DATABASE_ID=<<DATABASE_ID>>
VITE_APPWRITE_COLLECTION_ID=<<COLLECTION_ID>>
```
Σημείωση: Για TMDB χρησιμοποιείται Bearer Token (v4 API Read Access Token), όχι απλό API key σε query string.

### 5. Δομή Φακέλων
```
src/
	main.jsx          # Είσοδος / React root
	App.jsx            # Κύρια λογική UI & data fetching
	appwrite.js        # Λειτουργίες persistence (updateSearchCount, getTrendingMovies)
	components/
		Search.jsx       # Controlled input + debounce trigger
		MovieCard.jsx    # Παρουσίαση μίας ταινίας
		Spinner.jsx      # UI ένδειξη φόρτωσης
	assets/            # Στατικά αρχεία (svg κ.ά.)
public/              # Δημόσιες εικόνες (hero.png, no-movie.png, ...)
```

### 6. Εκτέλεση / Ανάπτυξη
```bash
npm install
npm run dev
```
Build παραγωγής:
```bash
npm run build
npm run preview
```

### 7. Μελλοντικές Ιδέες
- Προσθήκη pagination / infinite scroll.
- Detailed view (modal / route) ανά ταινία.
- User auth στο Appwrite για προσωποποιημένα favorites.
- Caching layer (IndexedDB ή service worker) για offline hints.

### 8. Πλεονεκτήματα χρήσης React (Σύντομο)
React προσφέρει component-based αρχιτεκτονική, καθαρή διαχείριση state με hooks και τεράστιο οικοσύστημα εργαλείων, επιταχύνοντας την ανάπτυξη διαδραστικών front-end εφαρμογών με επαναχρησιμοποιήσιμο κώδικα.

---

## 🇬🇧 English version

### 1. Description
**Movies Library** is a modern React Single Page Application (SPA) that lets users search for movies via the TMDB API while also tracking search popularity. Each search term is stored in an Appwrite database together with the first movie returned. Based on how often a term is searched, a dynamic Trending Movies list is generated. 

Key features:
- 🔍 Live-feel movie search with debounced input (fewer redundant API calls).
- ⭐ Display of core metadata (poster, rating, language, year).
- 📈 Dynamic Trending Movies derived from real user search frequency (`count`).
- 🛡️ Graceful failure handling (error messages, `no-movie.png` fallback).
- ⚡ Fast dev & HMR powered by Vite.

### 2. Architecture
The architecture is intentionally lightweight and UI/data-flow oriented:

| Layer | Description |
| ----- | ----------- |
| Presentation (React Components) | `App.jsx` as root container; supporting components: `Search`, `MovieCard`, `Spinner`. |
| Data Fetching | Simple `fetch` requests to TMDB API with Authorization Bearer token. |
| State Management | Local React hooks (`useState`, `useEffect`) + `useDebounce` from `react-use`. |
| Persistence Layer | Appwrite Database for storing/updating search terms and producing trending list. |
| Derivative Data | Trending movies built by ordering documents by `count` (descending) using Appwrite queries. |

#### Data Flow (High Level)
1. User types in `Search` component input.  
2. `searchTerm` updates instantly; `debouncedSearchTerm` updates after 300ms idle.  
3. Effect in `App.jsx` invokes `fetchMovies(debouncedSearchTerm)` (TMDB API).  
4. If there is a query & results, `updateSearchCount` runs → Appwrite: either creates a new document or increments `count`.  
5. On initial load `getTrendingMovies()` fetches Top 5 search-based entries.  
6. Two lists are rendered: Trending (from DB) & All Movies (current search or popular).  

#### Error Handling & Fallbacks
- Non-OK HTTP response → error message.
- Missing `poster_path` → `/no-movie.png` fallback.
- Empty or missing fields → safe defaults (e.g. `N/A`).

### 3. Technologies & Libraries
- **React 19**: Component-based UI with hooks.
- **Vite**: Dev server, HMR, optimized builds.
- **Tailwind CSS (v4)**: Utility-first styling (@tailwindcss/vite integration).
- **Appwrite SDK**: Client-side database operations (documents & queries).
- **TMDB API**: Source of movie metadata (title, poster, stats).
- **react-use (useDebounce)**: Simplifies search debounce logic.
- **ESLint**: Linting and code consistency.

### 4. Environment Variables (.env)
Create a `.env` (or `.env.local`) file at the project root:
```
VITE_TMDB_API_KEY=<<TMDB_BEARER_TOKEN>>
VITE_APPWRITE_ENDPOINT=https://cloud.appwrite.io/v1
VITE_APPWRITE_PROJECT_ID=<<PROJECT_ID>>
VITE_APPWRITE_DATABASE_ID=<<DATABASE_ID>>
VITE_APPWRITE_COLLECTION_ID=<<COLLECTION_ID>>
```
Note: For TMDB a Bearer Token (v4 API Read Access Token) is used—not a simple query string key.

### 5. Folder Structure
```
src/
	main.jsx          # Entry / React root
	App.jsx           # Main UI & data fetching logic
	appwrite.js       # Persistence helpers (updateSearchCount, getTrendingMovies)
	components/
		Search.jsx      # Controlled input + debounce trigger
		MovieCard.jsx   # Single movie presentation
		Spinner.jsx     # Loading indicator
	assets/           # Static assets (svg, etc.)
public/             # Public images (hero.png, no-movie.png, ...)
```

### 6. Run / Development
```bash
npm install
npm run dev
```
Production build:
```bash
npm run build
npm run preview
```

### 7. Future Ideas
- Pagination / infinite scroll.
- Detailed movie view (modal / route).
- User auth in Appwrite for personalized favorites.
- Caching layer (IndexedDB or service worker) for offline hints.

### 8. Advantages of React (Brief)
React provides a component-driven model, clean state management with hooks, and a vast ecosystem—speeding up the delivery of interactive front-end applications with reusable code.

---

