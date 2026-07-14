# Plan de marquage — Datacadémie (sandbox analytics)

**Page :** https://un-lucielafarge.github.io/Sandbox/test.html
**Dernière mise à jour :** 14 juillet 2026

## Outils en place

| Outil | Identifiant | Chargement |
|---|---|---|
| Google Tag Manager | `GTM-MP5X4V7D` | snippet dans le `<head>` |
| GA4 | via GTM | tags à configurer dans le conteneur |
| Matomo | siteId `1` (instance Clever Cloud) | snippet dans le `<head>` |
| PostHog | cloud EU (`eu.i.posthog.com`) | snippet dans le `<head>` — clé à renseigner ligne 54 de `test.html` |

Tous les événements ci-dessous sont envoyés **en parallèle** vers le `dataLayer` (→ GTM/GA4), PostHog et Matomo grâce au helper `track()`. Chaque envoi est aussi loggé en console (`[track]`) pour faciliter la recette.

## Convention de nommage

- Événements et paramètres en `snake_case`, en anglais.
- On réutilise les [événements recommandés GA4](https://support.google.com/analytics/answer/9267735) quand ils existent (`purchase`, `search`, `generate_lead`…).
- Deux événements legacy (`newsletterSubscription`, `productPurchase`) sont conservés en camelCase pour ne pas casser les déclencheurs GTM historiques.

## 1. Navigation & pages

| Événement | Déclencheur | Paramètres clés |
|---|---|---|
| _page view_ | chargement de page (natif GA4 / Matomo / PostHog) | dataLayer initial : `pageCategory`, `articleAuthor`, `articleLength`, `pageTags`, `userID`, `environment`, `publishDate` |
| `virtual_page_view` | clic sur « Essayer la démo interactive (SPA) » (`#push`, fait un `history.pushState`) | `virtual_page_path`, `virtual_page_title` |
| `menu_click` | clic sur un lien de la navigation principale | `link_text`, `link_url` |
| `search` | arrivée sur la page avec `?q=` (formulaire de recherche ou lien) | `search_term` (+ `trackSiteSearch` natif Matomo) |

## 2. E-commerce (format GA4)

| Événement | Déclencheur | Paramètres clés |
|---|---|---|
| `view_item_list` | chargement de la page | `ecommerce.item_list_name`, `ecommerce.items[]` (les 3 formations) |
| `select_item` | clic sur une carte formation | `ecommerce.items[]` (l'item cliqué) |
| `purchase` | clic sur « Valider l'achat du pack » (`#purchase`) | `ecommerce.transaction_id`, `value: 250`, `shipping: 5.99`, `currency: EUR`, `items[]` (+ ecommerce natif Matomo : `addEcommerceItem` × 2 + `trackEcommerceOrder`) |
| `productPurchase` *(legacy)* | même clic | ancien format `productPurchase.products[]` — conservé pour les déclencheurs GTM existants |

Chaque push e-commerce est précédé d'un `dataLayer.push({ ecommerce: null })` (bonne pratique GA4).

## 3. Conversion & leads

| Événement | Déclencheur | Paramètres clés |
|---|---|---|
| `generate_lead` | clic sur « S'abonner à la newsletter » (`#custom`) | `lead_source`, `email_domain` (jamais l'email complet — RGPD), `client_type`, `frequency` |
| `newsletterSubscription` *(legacy)* | même clic | ancien format — conservé pour GTM |
| `cta_click` | clic sur un élément portant `data-cta` (bandeau promo, CTA hero) | `cta_id`, `link_text`, `link_url` |

## 4. Engagement

| Événement | Déclencheur | Paramètres clés |
|---|---|---|
| `scroll_depth` | paliers 25 / 50 / 75 / 90 % (une fois chacun) | `percent_scrolled` |
| `section_view` | section visible à ≥ 20 % de l'écran (une fois par section) | `section_id` (`formations`, `mag`, `video`, `newsletter`) |
| `video_start` / `video_play` / `video_pause` / `video_complete` | lecteur YouTube (iframe API, `enablejsapi=1`) | `video_title`, `video_provider`, `video_url` |
| `outbound_click` | clic vers un domaine externe | `link_url`, `link_domain` |
| `file_download` | clic sur un lien `.pdf`, `.zip`, `.csv`, `.xlsx`, `.docx` | `file_name`, `file_extension`, `link_url` |

## 5. Zone de test (bas de page)

| Événement | Déclencheur | Effet |
|---|---|---|
| `userIdentify` | bouton « Identifier l'utilisateur » | `posthog.identify(uid)` + `setUserId` Matomo + push dataLayer `userId` |
| — | bouton « Déclencher une erreur JS » | lève une exception non catchée (pour tester l'error tracking PostHog / GTM) |
| `search` | lien « Recherche interne (?q=) » | recharge la page avec `?q=analytics` |

## Notes RGPD

- L'email saisi dans la newsletter n'est **jamais** envoyé aux outils : seul son domaine (`email_domain`) est collecté.
- PostHog est configuré en `person_profiles: 'identified_only'` : pas de profil utilisateur tant que `identify()` n'est pas appelé.
- Site fictif à but pédagogique — aucune donnée réelle.
