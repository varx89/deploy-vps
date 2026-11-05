# 🚀 Instalare Dokploy pe un VPS: Ghid complet

> 📘 **Scop:** transformarea unui singur VPS într-o platformă proprie de deploy pentru aplicații Docker, cu panou web, domenii proprii și HTTPS automat.
>

---

## 📑 Cuprins

1. [Prezentare generală](#-prezentare-generală)
2. [Cerințe preliminare](#-cerințe-preliminare)
3. [Pasul 1: Crearea VPS-ului](#️-pasul-1-crearea-vps-ului)
4. [Pasul 2: Conectarea prin SSH](#-pasul-2-conectarea-prin-ssh)
5. [Pasul 3: Pregătirea serverului](#️-pasul-3-pregătirea-serverului)
6. [Pasul 4: Firewall-ul din panoul furnizorului](#️-pasul-4-firewall-ul-din-panoul-furnizorului)
7. [Pasul 5: Firewall-ul UFW pe VPS](#-pasul-5-firewall-ul-ufw-pe-vps)
8. [Pasul 6: Instalarea Dokploy](#-pasul-6-instalarea-dokploy)
9. [Pasul 7: Configurarea DNS](#-pasul-7-configurarea-dns)
10. [Pasul 8: HTTPS pentru panoul Dokploy](#-pasul-8-https-pentru-panoul-dokploy)
11. [Pasul 9: Deploy din template (Kutt)](#-pasul-9-deploy-din-template-kutt)
12. [Pasul 10: Deploy din Git (WG-Easy)](#-pasul-10-deploy-din-repository-git-wg-easy)
13. [Pasul 11: Deploy din Docker Compose (Immich)](#-pasul-11-deploy-din-docker-compose-immich)
14. [Pasul 12: Actualizarea Dokploy](#-pasul-12-actualizarea-dokploy)
15. [Checklist final](#-checklist-final)
16. [Depanare rapidă](#-depanare-rapidă)

---

## 🧭 Prezentare generală

**Dokploy** este un panou de control web pentru instalarea și administrarea aplicațiilor Docker pe propriul server. Oferă o experiență similară cu Heroku, Vercel sau Railway, dar infrastructura rămâne sub controlul tău.

În acest ghid se instalează Dokploy și apoi trei aplicații, fiecare folosind o altă metodă de deploy:

| Aplicație | Rol | Metodă de deploy | Subdomeniu |
|---|---|---|---|
| 🔗 **Kutt** | Scurtare de linkuri (alternativă Bitly) | Template Dokploy | `go.exemplu.ro` |
| 🛡️ **WG-Easy** | Server VPN WireGuard cu dashboard | Repository Git public | `vpn.exemplu.ro` |
| 📸 **Immich** | Backup foto/video (alternativă Google Photos) | Docker Compose (Raw) | `photos.exemplu.ro` |
| 🎛️ **Dokploy** | Panoul de administrare | Script oficial | `dokploy.exemplu.ro` |

> 🔁 **Convenții:** în tot documentul, înlocuiește `exemplu.ro` cu domeniul tău și `IP_VPS` cu adresa IPv4 publică a serverului.

---

## ✅ Cerințe preliminare

| | Cerință | Detalii |
|---|---|---|
| 🌐 | **Domeniu** | Înregistrat la orice registrar, cu acces la administrarea DNS |
| 💻 | **Client SSH** | Terminal (macOS/Linux) sau Windows Terminal |
| 🖥️ | **VPS** | Ubuntu 24.04 LTS, curat, fără panou preinstalat |
| 🔑 | **Manager de parole** | Pentru parolele root, utilizator și administrator |

---

## 🖥️ Pasul 1: Crearea VPS-ului

### 📊 Resurse recomandate

| Nivel | CPU | RAM | Disc | Observații |
|---|---|---|---|---|
| Minim oficial | — | 2 GB | 30 GB | Suficient doar pentru teste ușoare |
| ⭐ Recomandat | 6 vCPU | 12 GB | 200 GB SSD | Loc pentru build-uri, baze de date și mai multe servicii |

> 💡 Dokploy este doar stratul de administrare. Serverul are nevoie de resurse și pentru sistemul de operare, containere, baze de date și loguri.

### ⚙️ Configurare la comandă

- 📍 **Regiune:** cea mai apropiată de tine sau de utilizatori
- 🐧 **Sistem de operare:** Ubuntu 24.04 LTS
- 🚫 **Panou de control / aplicație:** niciuna
- 🔐 **Autentificare:** parolă root lungă și unică, salvată în managerul de parole

> ⚡ **Variantă rapidă:** unii furnizori (ex. Contabo) oferă o imagine Dokploy „one-click”. Ghidul de față pornește de la un Ubuntu curat, pentru a înțelege ce rulează sub Dokploy și pentru a securiza serverul înainte de expunerea la internet.

📋 După creare, notează **adresa IPv4 publică** a serverului din panoul furnizorului.

---

## 🔌 Pasul 2: Conectarea prin SSH

```bash
ssh root@IP_VPS
```

1. La prima conectare, confirmă cheia serverului tastând `yes` și apasă **Enter**.
2. Introdu parola root.

> ℹ️ Este normal ca nimic să nu apară pe ecran în timp ce tastezi parola.

---

## 🛠️ Pasul 3: Pregătirea serverului

### 📦 Ce face scriptul

- 👤 Creează un utilizator normal (non-root) și îl adaugă în grupul `sudo`
- 🚫 Dezactivează autentificarea root cu parolă prin SSH
- 🔄 Actualizează Ubuntu și instalează utilitare de server
- 🧱 Instalează și activează **Fail2Ban**
- 🩹 Activează actualizările automate de securitate
- 🔥 Configurează **UFW** să blocheze implicit traficul de intrare
- 🐳 Instalează **Docker Engine** și **Docker Compose**

### ▶️ Rulare

```bash
curl -fsSL [[https://raw.githubusercontent.com/tonyflo/ttt-vps-scripts/main/setup-docker.sh](https://raw.githubusercontent.com/varx89/deploy-vps/refs/heads/main/setup-docker.sh?token=GHSAT0AAAAAAEOIP6HW4KLJ6TU2BQZFWYZE2XDLU6A)](https://raw.githubusercontent.com/varx89/deploy-vps/refs/heads/main/setup-docker.sh?token=GHSAT0AAAAAAEOIP6HW4KLJ6TU2BQZFWYZE2XDLU6A) | bash
```

1. Alege un nume de utilizator (litere mici), de exemplu `admin`.
2. Setează o parolă puternică; o vei folosi pentru comenzile `sudo`.
3. Serverul fie repornește automat, fie îți cere să închizi sesiunea root.
4. Reconectează-te cu noul utilizator:

```bash
ssh admin@IP_VPS
```

### 🧪 Verificare Docker

```bash
docker run --rm hello-world
```

✅ Trebuie să apară mesajul `Hello from Docker!`.

> 🔒 **Securitate suplimentară:** după acest pas serverul este mai sigur, dar merită aplicate și recomandările de hardening ale furnizorului de VPS.

---

## 🛡️ Pasul 4: Firewall-ul din panoul furnizorului

> ⚠️ **De ce un al doilea firewall?** Docker poate crea reguli de rețea care **ocolesc UFW** atunci când un container publică un port. Un firewall de rețea al furnizorului filtrează traficul *înainte* să ajungă la VPS.

### 🧩 Creare (exemplu Contabo)

1. **Network Services** → **Firewall**
2. **Create new firewall**
3. Nume: `dokploy-firewall`

### 📜 Reguli de intrare (inbound)

| Serviciu | Protocol | Port | Sursă | Durată |
|---|---|---|---|---|
| 🔑 SSH | TCP | `22` | **IP-ul tău public** | Permanent |
| 🌐 HTTP | TCP | `80` | Any | Permanent |
| 🔒 HTTPS | TCP | `443` | Any | Permanent |
| 🎛️ Dokploy setup | TCP | `3000` | **IP-ul tău public** | ⏳ Temporar |
| 🛡️ WireGuard VPN | UDP | `51820` | Any | Permanent |

Afli IP-ul tău public (al calculatorului de pe care te conectezi, nu al VPS-ului) cu:

```bash
curl -4 https://ifconfig.me
```

4. Deschide tab-ul **Active VPS/VDS**, asociază VPS-ul și salvează.

---

## 🔥 Pasul 5: Firewall-ul UFW pe VPS

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 51820/udp
sudo ufw status verbose
```

✅ Trebuie să vezi permise: **OpenSSH**, **80**, **443** și **51820/udp**.

> ℹ️ Portul `3000` **nu** se deschide în UFW: Docker îl expune oricum, iar accesul este limitat doar din firewall-ul furnizorului, la IP-ul tău, și se închide după configurare.

---

## 📥 Pasul 6: Instalarea Dokploy

```bash
curl -sSL https://dokploy.com/install.sh | sudo sh
```

Installer-ul detectează Docker-ul existent și configurează **Dokploy**, **Docker Swarm**, **Traefik** și serviciile auxiliare.

### 🧪 Verificare servicii

```bash
docker service ls
```

✅ Serviciul `dokploy` și serviciile sale auxiliare trebuie să fie listate și pornite.

> ⏸️ Nu crea încă contul de administrator pe IP; mergi întâi la configurarea DNS (Pasul 7), apoi creezi contul pe subdomeniu (Pasul 8).

---

## 🌍 Pasul 7: Configurarea DNS

Creează următoarele înregistrări **A** la registrar, toate către `IP_VPS`:

| Host | Tip | Valoare | Aplicație |
|---|---|---|---|
| `dokploy` | A | `IP_VPS` | 🎛️ Panou Dokploy |
| `go` | A | `IP_VPS` | 🔗 Kutt |
| `vpn` | A | `IP_VPS` | 🛡️ WG-Easy |
| `photos` | A | `IP_VPS` | 📸 Immich |

> ☁️ **Cloudflare:** setează toate înregistrările pe **DNS only** (norul gri) pe durata configurării, altfel emiterea certificatelor Let’s Encrypt și traficul VPN pot avea probleme.

🧪 Verifică propagarea:

```bash
dig +short dokploy.exemplu.ro
```

---

## 🔒 Pasul 8: HTTPS pentru panoul Dokploy

### 1️⃣ Creare cont administrator

Deschide în browser:

```
http://dokploy.exemplu.ro:3000/register
```

Creează contul de administrator cu o parolă puternică și unică.

### 2️⃣ Setare domeniu și certificat

**Settings** → **Web Server**:

| Câmp | Valoare |
|---|---|
| Domain | `dokploy.exemplu.ro` |
| HTTPS | ✅ Activat |
| Certificate Provider | Let’s Encrypt |

Salvează și așteaptă câteva momente ca Traefik să obțină certificatul. Testează într-un tab nou:

```
https://dokploy.exemplu.ro
```

### 3️⃣ Închiderea portului 3000

După ce panoul funcționează pe HTTPS, elimină publicarea portului 3000:

```bash
docker service update \
  --publish-rm "published=3000,target=3000,mode=host" \
  dokploy
```

🧪 `http://IP_VPS:3000` nu mai trebuie să se încarce.

4️⃣ În firewall-ul furnizorului, **șterge regula temporară pentru TCP 3000**.

### 🎯 Rezultat

| Port | Rol |
|---|---|
| `80` | HTTP și validarea certificatelor |
| `443` | Trafic HTTPS securizat |

---

## 🔗 Pasul 9: Deploy din template (Kutt)

**Kutt** este un serviciu open-source de scurtare a linkurilor, cu domeniu propriu și statistici, fără abonament.

### 📝 Pași

1. **Create Project** → nume: `Self-Hosted Apps`
2. **Create Service** → **Template**
3. Caută **Kutt**
4. În **Environment**:

   ```env
   DEFAULT_DOMAIN=go.exemplu.ro
   TRUST_PROXY=true
   ```

   > 💡 `TRUST_PROXY=true` permite citirea IP-ului real al vizitatorului din headerele proxy-ului, pentru statistici mai precise (inclusiv țara).

5. În **Domains**: Host `go.exemplu.ro`, HTTPS activat cu Let’s Encrypt
6. Tab-ul **General** → **Deploy**

> ⚙️ Template-ul generează automat secretul JWT, stocarea persistentă pentru baza de date și rutarea către portul intern 3000.

### 🧪 Test

Deschide `https://go.exemplu.ro`, creează contul de administrator și scurtează un link de probă.

---

## 🛡️ Pasul 10: Deploy din repository Git (WG-Easy)

**WG-Easy** oferă un dashboard web pentru un server VPN WireGuard propriu. Se folosește o variantă a fișierului Compose adaptată pentru Dokploy, care separă rețeaua VPN de rutarea dashboard-ului.

### 🧩 Creare serviciu

1. În proiectul `Self-Hosted Apps`: **Create Service** → **Compose**
2. **Name** și **App Name**: `wg-easy`
3. **Create**

### 📂 Provider: Git

| Câmp | Valoare |
|---|---|
| Repository URL | `https://github.com/tonyflo/wg-easy-dokploy.git` |
| Branch | `main` |
| Compose Path | `./docker-compose.yml` |

> ℹ️ Pentru un repository public se poate folosi **Git** direct, fără conectarea contului GitHub. Provider-ul **GitHub** este util pentru repository-uri private.

### 🌐 Domeniu

| Câmp | Valoare |
|---|---|
| Service Name | `wg-easy` |
| Host | `vpn.exemplu.ro` |
| Container Port | `51821` |
| HTTPS | ✅ Let’s Encrypt |

> 🔄 Dacă `wg-easy` nu apare în lista de servicii, apasă refresh. Numele trebuie să corespundă celui din fișierul Compose.

| Port | Protocol | Rol |
|---|---|---|
| `51820` | UDP | Tunel WireGuard (public) |
| `51821` | TCP | Dashboard web (intern, prin Traefik) |

### 🚀 Deploy

**General** → **Deploy**. Generarea certificatului poate dura câteva minute. Deschide `https://vpn.exemplu.ro` și completează wizard-ul:

- **Host:** `vpn.exemplu.ro`
- **Port WireGuard:** `51820`

### 📱 Conectarea primului dispozitiv

1. Creează un client nou (ex. `Telefon personal`)
2. Descarcă fișierul de configurare
3. Instalează aplicația oficială **WireGuard**
4. Importă configurația

### 🧪 Test

1. Verifică IP-ul curent (ex. [ipinfo.io/ip](https://ipinfo.io/ip)) **înainte** de activarea tunelului
2. Activează tunelul
3. Verifică din nou: trebuie să apară **IP-ul VPS-ului**
4. În WG-Easy, confirmă că apare trafic transferat

---

## 📸 Pasul 11: Deploy din Docker Compose (Immich)

**Immich** este o platformă de backup și organizare foto/video cu aplicații mobile, backup automat, recunoaștere facială, căutare, albume și partajare.

### 🧩 Creare serviciu

1. **Create Service** → **Compose**
2. **Name** și **App Name**: `immich`
3. **Compose Type**: Docker Compose → **Create**
4. Deschide serviciul → **General** → **Provider**: **Raw**

### 📄 Fișierul Compose

1. Deschide [ultima versiune Immich pe GitHub](https://github.com/immich-app/immich/releases/latest)
2. Click pe tag-ul versiunii → folderul `docker` → `docker-compose.yml` → **Copy raw file**
3. Lipește conținutul în editorul **Compose File** din Dokploy

> ⚠️ Avertismentele despre ramura `main` (pe GitHub și în comentariul din fișier) pot fi ignorate, deoarece fișierul provine dintr-o versiune etichetată (release).

4. **Șterge** maparea de port, deoarece rutarea se face intern prin Dokploy:

```yaml
ports:
  - '2283:2283'
```

### 🧱 Servicii incluse

| Serviciu | Rol |
|---|---|
| 🖥️ Immich Server | Aplicația web |
| 🧠 Machine Learning | Recunoaștere facială și căutare inteligentă |
| ⚡ Redis | Coordonarea job-urilor din fundal |
| 🐘 PostgreSQL | Utilizatori, albume, metadate |

### 🔧 Variabile de mediu

Din același folder `docker`, copiază `example.env` în tab-ul **Environment** și modifică:

```env
TZ=Europe/Bucharest
DB_PASSWORD=INLOCUIESTE_CU_O_PAROLA_ALEATOARE
```

> 🔐 Generează o parolă sigură: `openssl rand -base64 24 | tr -d '/+='`

### 🌐 Domeniu

| Câmp | Valoare |
|---|---|
| Domain | `photos.exemplu.ro` |
| Service | `immich-server` |
| Container Port | `2283` |
| HTTPS | ✅ Let’s Encrypt |

### 🚀 Deploy

**General** → **Deploy**, apoi urmărește progresul în **Deployments**. Când toate serviciile rulează, deschide `https://photos.exemplu.ro` și creează contul.

> 💾 **Backup obligatoriu:** Immich nu trebuie să fie singura copie a fotografiilor. Fă backup atât pentru fișierele media, cât și pentru baza de date PostgreSQL ([documentația oficială](https://docs.immich.app/administration/backup-and-restore/)).

---

## 🔄 Pasul 12: Actualizarea Dokploy

```bash
curl -sSL https://dokploy.com/install.sh | sudo sh -s update
```

> ⚠️ Înainte de actualizări majore, asigură-te că ai backup-uri recente pentru aplicațiile și bazele de date importante.

---

## 📋 Checklist final

- [ ] VPS cu Ubuntu 24.04 LTS creat
- [ ] Utilizator non-root creat, login root cu parolă dezactivat
- [ ] Fail2Ban și actualizări automate active
- [ ] Docker funcțional (`hello-world`)
- [ ] Firewall furnizor configurat și asociat VPS-ului
- [ ] UFW: 80, 443, 51820/udp permise
- [ ] Dokploy instalat, servicii pornite
- [ ] Înregistrări DNS A create (`dokploy`, `go`, `vpn`, `photos`)
- [ ] Panou Dokploy pe HTTPS
- [ ] Portul 3000 închis (serviciu Docker + firewall furnizor)
- [ ] Kutt funcțional pe `go.exemplu.ro`
- [ ] WG-Easy funcțional, dispozitiv conectat, IP verificat
- [ ] Immich funcțional pe `photos.exemplu.ro`
- [ ] Strategie de backup stabilită

---

## 🧯 Depanare rapidă

| Problemă | Cauză probabilă | Soluție |
|---|---|---|
| ❌ Certificatul HTTPS nu se emite | DNS nepropagat sau proxy Cloudflare activ | Verifică cu `dig`, setează **DNS only**, așteaptă și redeploy |
| ❌ Panoul nu se deschide pe `:3000` | Regula temporară lipsește din firewall | Adaugă TCP 3000 doar pentru IP-ul tău |
| ❌ Serviciul nu apare la Domains | Lista nu s-a actualizat | Apasă refresh; numele trebuie să corespundă Compose-ului |
| ❌ VPN-ul se conectează, dar nu e trafic | UDP 51820 blocat | Verifică UFW și firewall-ul furnizorului |
| ❌ Immich nu pornește | Variabile `.env` incomplete | Verifică `DB_PASSWORD` și logurile din **Deployments** |
| 🔍 Diagnostic general | — | `docker service ls`, `docker ps`, logurile din Dokploy |

---

<sub>📄 Documentație adaptată în limba română după tutorialul publicat de Tony Teaches Tech. Comenzile și configurațiile aparțin proiectelor open-source menționate (Dokploy, Kutt, WG-Easy, Immich).</sub>
