# UTM Builder · Zoner Studio

Interní nástroj pro generování UTM parametrů podle firemní dokumentace.

## Použití

Otevřete soubor `index.html` přímo v prohlížeči – žádná instalace, žádný server.

## Jak na to

1. Zadejte cílovou URL webu
2. Vyberte typ kanálu (medium)
3. Doplňte zdroj a případně kampaň
4. Klikněte **Kopírovat URL**

## Kanály

| Kanál | utm_medium |
|---|---|
| Aplikace Zoner Studio | `application` |
| Placená reklama | `cpc` |
| Organické sociální sítě | `social` |
| Affiliate / Ambasadoři | `affiliate` |
| Placené zmínky | `sponsored_content` |
| Organické PR | `referral` |
| Vlastní platformy | `owned_media` |

> **Mailing** není zahrnut – UTM parametry pro emailing generuje automaticky CRM.

## Zlatá pravidla (GA4)

- Vždy malá písmena
- Žádné mezery (nahrazeny `_`)
- Žádná diakritika
- UTM jen na odkazech na produktový web

## Technické detaily

Jeden self-contained HTML soubor bez externích závislostí. Uložení partnerů
(affiliate, referral, sponsored_content) probíhá přes `localStorage` prohlížeče.

Pravidla jsou zakódována v objektu `RULES` v souboru `index.html`.
