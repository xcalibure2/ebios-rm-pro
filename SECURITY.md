# 🔒 Sécurité - EBIOS RM Pro v5

## Vue d'ensemble

EBIOS RM Pro v5 est une application web **100% client-side** (exécutée uniquement dans le navigateur). Les données sont stockées localement dans le `localStorage` du navigateur et ne sont jamais transmises à un serveur distant.

## Mesures de sécurité implémentées

### 1. Protection contre les injections XSS

Toutes les données utilisateur sont **échappées** avant affichage via la fonction `esc()` :

```javascript
function esc(str){
    if(str===null||str===undefined)return '';
    const div=document.createElement('div');
    div.textContent=String(str);
    return div.innerHTML;
}
```

### 2. Validation et sanitization des entrées

Les entrées utilisateur sont validées et nettoyées :

- **Longueur maximale** : Les chaînes sont tronquées (200-1000 caractères selon le champ)
- **Caractères dangereux** : Les caractères `<` et `>` sont filtrés
- **Détection de patterns malveillants** : Recherche de `<script>`, `javascript:`, `on*=`, etc.

```javascript
function sanitizeStr(str, maxLen=500){
    if(typeof str!=='string')return '';
    return str.substring(0,maxLen).replace(/[<>]/g,'');
}

function hasInjection(str){
    if(typeof str!=='string')return false;
    const patterns=[/<script/i,/javascript:/i,/on\w+=/i,/data:/i];
    return patterns.some(p=>p.test(str));
}
```

### 3. Validation des imports JSON

Lors de l'import d'un fichier JSON :

- **Taille maximale** : 5 Mo
- **Validation du schéma** : Vérification de la structure attendue
- **Détection d'injections** : Analyse du contenu pour détecter les patterns malveillants

```javascript
function validateJSON(data){
    if(!data||typeof data!=='object')throw new Error('Format JSON invalide');
    // Vérification de la structure...
    // Nettoyage des données...
    return data;
}
```

### 4. Content Security Policy (CSP)

L'application inclut une CSP restrictive :

```html
<meta http-equiv="Content-Security-Policy" content="
    default-src 'self'; 
    script-src 'self' 'unsafe-inline' 'unsafe-eval' https://cdn.tailwindcss.com https://cdn.jsdelivr.net; 
    style-src 'self' 'unsafe-inline' https://cdn.tailwindcss.com; 
    img-src 'self' data: blob:; 
    font-src 'self' https://fonts.gstatic.com; 
    connect-src 'self';
">
```

### 5. Subresource Integrity (SRI)

Les scripts CDN sont chargés avec des hashes d'intégrité :

```html
<script src="https://cdn.jsdelivr.net/npm/docx@8.5.0/build/index.umd.min.js" 
        integrity="sha384-..." 
        crossorigin="anonymous"></script>
```

## Stockage des données

- **Localisation** : `localStorage` du navigateur (clé `ebios5`)
- **Aucune transmission réseau** : Les données restent sur la machine de l'utilisateur
- **Export** : Fichiers JSON téléchargés localement

## Bonnes pratiques pour les utilisateurs

1. **Utilisez un navigateur à jour** : Chrome, Firefox, Edge dernières versions
2. **Ne partagez pas vos exports JSON** contenant des données sensibles
3. **Vérifiez la source** des fichiers JSON avant de les importer
4. **Effacez le localStorage** si vous utilisez un ordinateur partagé

## Signalement de vulnérabilités

Si vous découvrez une faille de sécurité :

1. **Ne la divulguez pas publiquement**
2. Contactez-moi via [créer une issue privée sur GitHub]
3. Fournissez un maximum de détails pour reproduire le problème

## Limitations connues

- L'application nécessite `'unsafe-inline'` et `'unsafe-eval'` pour Tailwind CSS et les templates
- Le localStorage n'est pas chiffré (limitation navigateur)
- Pas d'authentification utilisateur (application locale)

## Changelog sécurité

### v5.0
- Ajout de la fonction `esc()` pour l'échappement HTML
- Validation des imports JSON avec schéma
- Sanitization de toutes les entrées utilisateur
- Détection des patterns d'injection
- Ajout des headers CSP
- Subresource Integrity pour les CDN

---

*Dernière mise à jour : Janvier 2026*
