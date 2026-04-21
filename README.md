# SUBHUNTER 🔍

> **Framework modular de recon ofensivo para Bug Bounty e Pentest externo.**  
> Cobre todo o ciclo de ataque: da seed inicial até fuzzing ativo e extração de segredos.

---

## Visão Geral

O SUBHUNTER é um framework em Bash que orquestra mais de 40 ferramentas do ecossistema de segurança ofensiva em um pipeline coeso e executável com um único comando. Cada módulo pode ser executado de forma independente ou como parte do pipeline completo (`recon all`).

O projeto foi construído para uso real em programas de Bug Bounty e testes de pentest externo, com atenção a OPSEC (modo stealth), tratamento de wildcards DNS, e consolidação de resultados entre etapas.

---

## Pipeline de Execução

```
[SEED]
  └── root          → Extrai root domains via crt.sh
  └── reversewhois  → Expande escopo via WHOIS reverso (Whoxy API)

[ASN + INFRA]
  └── asn           → Mapeia ASNs e ranges de IP da organização
  └── amsanintel    → Amass intel via ASN + WHOIS
  └── shodan        → Busca hosts expostos via Shodan/Censys/FOFA
  └── cidr          → Expande CIDRs e executa PTR reverso

[INTEL PASSIVA]
  └── dnsrecon      → Zone transfer, DNSSEC/NSEC walking, enumLDAP
  └── mailrecon     → MX/TXT/DMARC + extração de IOCs
  └── csp           → Extração de domínios via Content-Security-Policy
  └── dorks         → Google Dorks automatizados (googler/pagodo)
  └── github        → Subdomínios vazados + secrets (TruffleHog/Gitleaks/CFOR)
  └── uncover       → Multi-engine: Shodan, Censys, FOFA, Hunter
  └── cloud         → Cloud asset discovery (AWS/GCP) + S3 misconfiguration

[ENUMERAÇÃO PASSIVA]
  └── subs          → Subfinder, Assetfinder, Findomain, Amass, Chaos, BBOT
                      + crt.sh, Wayback CDX, URLScan, GitHub code search

[RESOLUÇÃO + BRUTEFORCE]
  └── smartenum     → Geração inteligente de wordlist por padrões
  └── bruteforce    → PureDNS com wildcard detection e rate control
  └── permutations  → Alterx/dnsgen/gotator para variações de subdomínios
  └── validate      → Merge e resolução DNS final com PureDNS/dnsx

[RECON ATIVO]
  └── extractips    → Extração de IPs únicos
  └── reverseip     → Reverse IP lookup (hakip2host)
  └── live          → Filtragem de hosts ativos com httpx
  └── takeover      → Subdomain takeover (Nuclei templates + Subzy)
  └── portscan      → Masscan + Nmap com fingerprinting de serviços
  └── httpxfull     → HTTP probe completo: WAF, CDN, TLS, titles, techs
  └── screenshot    → Captura de telas com EyeWitness

[ANÁLISE DE CONTEÚDO]
  └── crawljs       → Katana + Gospider + GAU + Wayback para URL harvesting
  └── params        → Descoberta de parâmetros com Arjun
  └── jssecrets     → Extração de segredos em JS (Nuclei + SecretFinder + jsluice)

[FUZZING ATIVO]
  └── buildsubs     → Consolida wordlist de subdomínios para fuzzing
  └── buildpaths    → Consolida wordlist de paths para fuzzing
  └── fuzz          → FFUF massivo em subdomínios × paths
  └── kiterunner    → Descoberta de rotas de API com Kiterunner

[PIPELINE COMPLETO]
  └── all           → Executa todas as etapas em sequência
```

---

## Instalação

```bash
git clone https://github.com/c0st40/subhunter
cd subhunter
chmod +x recon
sudo mv recon /usr/local/bin/recon
```

### Diagnóstico de dependências

```bash
recon tools
```

Verifica quais ferramentas obrigatórias e opcionais estão instaladas, exibindo versões das principais.

---

## Uso

### Pipeline completo
```bash
recon all exemplo.com
recon all exemplo.com --mode stealth
recon all dominios.txt --force-resolve
```

### Módulos individuais

```bash
# Seed
recon root exemplo.com
recon reversewhois exemplo.com

# Infra
recon asn root_domains.txt asninfo.txt
recon shodan exemplo.com --org "Example Corp" --asn-file asninfo.txt
recon cidr exemplo.com --asn-file asninfo.txt

# Intel passiva
recon dnsrecon exemplo.com
recon mailrecon exemplo.com
recon csp exemplo.com
recon github exemplo.com --org exemplo --token "$GITHUB_TOKEN"
recon cloud exemplo.com --keyword exemplo

# Enumeração passiva
recon subs exemplo.com --mode stealth

# Resolução
recon bruteforce exemplo.com --wordlist smart_wordlist.txt
recon validate --passive all_subs.txt --brute brute_raw.txt --force-resolve

# Recon ativo
recon portscan --hosts resolved.txt --ips resolved_ips.txt
recon httpxfull --input resolved.txt
recon takeover --input all_subs_merged.txt

# Análise de conteúdo
recon crawljs --live live.txt
recon jssecrets --urls all_urls.txt
recon params --urls all_urls.txt

# Fuzzing
recon fuzz exemplo.com --subs subdomains.txt --paths paths.txt
recon kiterunner --live live.txt
```

---

## Variáveis de Ambiente

| Variável | Módulo | Descrição |
|---|---|---|
| `GITHUB_TOKEN` | github, subs | Token de acesso à API do GitHub |
| `SHODAN_API_KEY` | shodan | Chave da API Shodan |
| `WHOXY_API_KEY` | reversewhois | Chave da API Whoxy para WHOIS reverso |
| `SUBHUNTER_NO_BANNER` | global | Desativa o banner de inicialização |

---

## Ferramentas Integradas

**Enumeração passiva:** subfinder, assetfinder, findomain, amass, chaos, bbot  
**Resolução DNS:** puredns, dnsx, dnsgen, alterx, altdns, gotator  
**Recon ativo:** httpx, naabu, masscan, nmap, hakip2host  
**Web crawling:** katana, gospider, gau, waybackurls, waymore, uro  
**Fuzzing:** ffuf, feroxbuster, kiterunner (kr)  
**Secrets/OSINT:** trufflehog, gitleaks, nuclei, jsluice, SecretFinder, mantra, gf  
**Cloud/Infra:** asnmap, mapcidr, cloudbrute, s3scanner, uncover, shodan CLI  
**Misc:** wafw00f, cdncheck, testssl.sh, EyeWitness, subzy, cvemap  

---

## Outputs Gerados

| Arquivo | Conteúdo |
|---|---|
| `root_domains.txt` | Root domains da organização |
| `all_subs.txt` | Todos os subdomínios coletados passivamente |
| `resolved.txt` | Subdomínios com DNS resolvido |
| `live.txt` | Hosts com HTTP/HTTPS ativo |
| `httpx_full.txt` | Probe HTTP completo com tecnologias, status, WAF |
| `nmap_services.xml` | Resultados de port scan com fingerprinting |
| `all_urls.txt` | URLs coletadas via crawling |
| `js_exposures.txt` | Exposições encontradas em arquivos JS |
| `secrets.txt` | Segredos extraídos de JS e repositórios |
| `takeover_findings.txt` | Candidatos a subdomain takeover |
| `github_secrets.json` | Segredos encontrados via TruffleHog |
| `cloud_findings.txt` | Assets cloud descobertos |
| `s3_findings.txt` | Buckets S3 e misconfigurations |

---

## Modos de Operação

| Modo | Threads | Uso |
|---|---|---|
| `normal` | Alto (100+) | Ambientes de lab, programas sem restrição de rate |
| `stealth` | Baixo (3-25) | Programas com WAF/rate limiting agressivo |

---

## Notas de OPSEC

- O modo `--stealth` reduz threads e cadência de requisições para minimizar detecção por WAF/IDS
- O módulo `dnsrecon` detecta automaticamente DNSSEC e executa NSEC walking apenas quando aplicável
- Todos os módulos possuem fallback gracioso: ferramentas ausentes são reportadas e puladas sem abortar o pipeline
- Wildcards DNS são detectados e tratados nos módulos de bruteforce para evitar falsos positivos massivos

---

## Autor

**Daniel Costa**  
[GitHub](https://github.com/c0st40) | [LinkedIn](https://linkedin.com/in/c0st40)  
Bug Bounty | Pentest | Security Automation

---

> ⚠️ **Uso ético.** Este framework é destinado exclusivamente a atividades autorizadas: programas de Bug Bounty com escopo definido, ambientes de laboratório e testes de penetração contratados. O uso não autorizado contra sistemas de terceiros é ilegal.
