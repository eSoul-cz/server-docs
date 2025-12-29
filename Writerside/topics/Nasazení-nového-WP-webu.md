# Nasazení nového WP webu

Instalace WP webu je zatím semi-automatizovaná pomocí skriptu, ale pořád vyžaduje několik manuálních kroků.

## Postup instalace:

### 1. Příprava databáze

Přes ()[Adminer.md] (přihlášení na `root` uživatele) je třeba vytvořit novou databázi a uživatele s přístupem k té databázi.

1. Vytvořte novou databázi, např. `example_db`.
2. Vytvořte nového uživatele, např. `example_user` (pro server `%`), a nastavte mu heslo, např. `example_pass`.
3. Udělte tomuto uživateli přístup k nově vytvořené databázi.

### 2. Příprava adresáře pro aplikaci

Všechny soubory aplikací jsou umístěny v adresáři `/data/apps`.

```bash
cd /data/apps
```

Adresářová struktura aplikací by měla odpovídat: `/data/apps/{klient}/{aplikace}`. Takže pro klienta `example` vytvoříme adresář:

```bash
mkdir -p /data/apps/example
```

> Adresář pro aplikaci není třeba vytvářet ručně, instalační skript ho vytvoří automaticky.

Práva k adresáři by měli být nastavena na skupinu `apps` a uživatele `root`:

```bash
sudo chown -R root:apps /data/apps/example
```

### 3. Instalace WordPress webu

Nyní můžeme v adresáři spustit instalační skript pro WordPress:

```bash
cd /data/apps/example
sudo wp-bootstrap 
  --repo=https://github.com/eSoul-cz/WP-project-template \
  --project-name="Example Web" \
  --db-name="example_db" \ 
  --db-user="example_user" \ 
  --db-pass="example_pass" \
  --domain=example.cz \
  -a admin:admin@esoul.cz
  -a admin2:admin2@esoul.cz
```

#### Argumenty:

- `--repo`: URL Git repozitáře s WordPress šablonou - vždy bude ()[https://github.com/eSoul-cz/WP-project-template].
- `--project-name`: Název projektu (webu). Z toho se odvodí název adresáře.
- `--db-name`: Název databáze pro WordPress.
- `--db-user`: Uživatelské jméno pro přístup k databázi.
- `--db-pass`: Heslo pro přístup k databázi.
- `--domain`: Hlavní doména webu.
- `-a`: Přidání administrátorského uživatele ve formátu `uživatel:email`. Tento argument lze použít vícekrát pro přidání více uživatelů. Výstupem scriptu budou vygenerovaná hesla pro tyto uživatele.
- `--ref`: (volitelný) Git větev nebo tag, který se má použít. Výchozí je `master`.
- `--force`: (volitelný) Přepíše existující instalaci, pokud již adresář s daným názvem existuje.
- `--reuse-existing`: (volitelný) Nebude znovu stahovat repozitář, pokud už byl stažený.
- `--overwrite`: (volitelný) Před stažením repozitáře smaže existující soubory v cílovém adresáři.
- `--target-dir`: (volitelný) Cílový adresář pro instalaci. Pokud není zadán, použije se `./{project-name}`.
- `--no-db-import`: (volitelný) Přeskočí import databáze.
- `--db-host`: (volitelný) Hostitel databáze, výchozí je `host.docker.local` (není třeba měnit).
- `--db-port`: (volitelný) Port databáze, výchozí je `3306` (není třeba měnit).

### 4. Nastavení dockeru

V ()[Komodo.md] je třeba vytvořit nový stack. Pro zjednodušení lze použít šablonu `wp-template`.

1. Přihlašte se do Komodo UI.
2. V menu přejděte na "Stacks" a klikněte na "New Stack".
3. Zadejte název stacku, např. `Example web`.
4. Vyberte šablonu `wp-template`.
5. V sekci `Config` je třeba upravit následující:
    - `Run Directory`: `/data/apps/klient/app` (cesta k aplikaci - vytvořená pomocí instalačního skriptu).
6. Uložite stack a spusťte ho kliknutím na "Deploy".

### 5. Nastavení Nginx

V ()[Nginx-UI.md] je třeba vytvořit novou doménu pro web.

1. Přihlašte se do Nginx UI.
2. V menu přejděte na "Manage Sites" a klikněte na "Sites List".
3. Pokud proběhl instalační skript správně, v seznamu už by měla být vytvořená doména podle definice v kroku 3 (`example.cz`).
4. Klikněte u domény na "Edit".
5. V pravém panelu přepněte `Status` na `Enabled` a uložte.

Teď už by měl web být dostupný, ale zatím jen na HTTP - některé funkce nebudou fungovat správně, dokud nebude nastavený SSL certifikát.

### 6. Nastavení SSL certifikátu

V ()[Scaleway.md] je třeba nastavit SSL certifikát pro doménu.

1. Přihlašte se do Scaleway Console.
2. V sekci `Network` přejděte na `Load Balancers`.
3. Vyberte load balancer, který obsluhuje váš web server (Region `PAR 2` - `dedibox-balancer`).
4. Přejděte na záložku `SSL Certificates` a klikněte na `Create SSL Certificate`.
5. Název i `Common Name` nastavte na vaši doménu (`example.cz` - bez `https://`).
    - Pokud je třeba, přidejte i `Subject Alternative Names` (např. `www.example.cz`).
6. Vyberte `Let's Encrypt` jako poskytovatele certifikátu.
7. Vytvořte certifikát kliknutím na `Create SSL Certificate`.
8. Přejděte na záložku `Frontends` a upravte frontend pro HTTPS port 443 (`esoul https`):
9. Klikněte na `Edit`.
10. V sekci `SSL Certificate(s)` přidejte do výběru nově vytvořený certifikát.
11. Uložte změny kliknutím na `Edit Frontend`.

## Konfigurace

Po dokončení výše uvedených kroků by měl být WordPress web plně funkční a dostupný přes HTTPS.
Konfigurace webu se může dělat přes ()[Komodo.md] v nastavení stacku v sekci `Info`. 

### Vypnutí WP_DEBUG

Ve výchozím stavu je zapnutý WP_DEBUG. Pro vypnutí je třeba editovat `WORDPRESS_DEBUG=` (na prázdnou hodnotu) v `.env` souboru.

Editaci lze provést přes Komodo UI:

1. Přejděte do Komodo UI.
2. V menu přejděte na "Stacks" a vyberte váš stack (např. `Example web`).
3. Klikněte na záložku "Info".
4. Najděte soubor `.env` v sekci "Files" a klikněte na něj.
5. Najděte řádek `WORDPRESS_DEBUG=true` a změňte ho na `WORDPRESS_DEBUG=`.
6. Uložte změny kliknutím na "Save".
7. Restartujte stack kliknutím na "Redeploy".