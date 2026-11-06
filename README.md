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
11. [Pasul 9: Actualizarea Dokploy](#-pasul-9-actualizarea-dokploy)
12. [Checklist final](#-checklist-final)
13. [Depanare rapidă](#-depanare-rapidă)

---

## 🧭 Prezentare generală

**Dokploy** este un panou de control web pentru instalarea și administrarea aplicațiilor Docker pe propriul server. Oferă o experiență similară cu Heroku, Vercel sau Railway, dar infrastructura rămâne sub controlul tău.

În acest ghid se instalează și se securizează Dokploy, pregătit pentru deploy-ul aplicațiilor:

| Aplicație | Rol | Metodă de deploy | Subdomeniu |
|---|---|---|---|
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
curl -fsSL https://raw.githubusercontent.com/varx89/deploy-vps/refs/heads/main/setup-docker.sh | sudo bash
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
sudo ufw status verbose
```

✅ Trebuie să vezi permise: **OpenSSH**, **80** și **443**.

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

Creează următoarea înregistrare **A** la registrar, către `IP_VPS`:

| Host | Tip | Valoare | Aplicație |
|---|---|---|---|
| `dokploy` | A | `IP_VPS` | 🎛️ Panou Dokploy |

> ☁️ **Cloudflare:** setează înregistrarea pe **DNS only** (norul gri) pe durata configurării, altfel emiterea certificatelor Let’s Encrypt poate avea probleme.

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

## 🔄 Pasul 9: Actualizarea Dokploy

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
- [ ] UFW: 80, 443 permise
- [ ] Dokploy instalat, servicii pornite
- [ ] Înregistrare DNS A creată (`dokploy`)
- [ ] Panou Dokploy pe HTTPS
- [ ] Portul 3000 închis (serviciu Docker + firewall furnizor)
- [ ] Strategie de backup stabilită

---

## 🧯 Depanare rapidă

| Problemă | Cauză probabilă | Soluție |
|---|---|---|
| ❌ Certificatul HTTPS nu se emite | DNS nepropagat sau proxy Cloudflare activ | Verifică cu `dig`, setează **DNS only**, așteaptă și redeploy |
| ❌ Panoul nu se deschide pe `:3000` | Regula temporară lipsește din firewall | Adaugă TCP 3000 doar pentru IP-ul tău |
| ❌ Serviciul nu apare la Domains | Lista nu s-a actualizat | Apasă refresh; numele trebuie să corespundă Compose-ului |
| 🔍 Diagnostic general | — | `docker service ls`, `docker ps`, logurile din Dokploy |

---

<sub>📄 Documentație adaptată în limba română după tutorialul publicat de Tony Teaches Tech. Comenzile și configurațiile aparțin proiectelor open-source menționate (Dokploy).</sub>
