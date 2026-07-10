## Homelab Architecture

Środowisko opiera się na hypervisorze **Proxmox VE**. Za **routing**, **firewall** oraz segmentację sieci odpowiada **pfSense**. Dostęp zdalny do środowiska realizowany jest poprzez tunel **VPN** z wykorzystaniem protokołu **WireGuard**.

### Network Segmentation
Sieć została podzielona na odizolowane podsieci, zmapowane do odpowiednich interfejsów **bridge** w Proxmox:

*   **VPN** (`10.10.0.0/24`) – szyfrowany dostęp z zewnątrz (**WireGuard**).
*   **LAN** (`10.20.0.0/24` | `vmbr1`) – infrastruktura zarządzająca, monitoring oraz narzędzia wewnętrzne:
    *   **Ansible** (automatyzacja i konfiguracja)
    *   **Wazuh** (SIEM / XDR do monitorowania bezpieczeństwa)
    *   **Portainer** (zarządzanie środowiskiem kontenerowym)
    *   **Vikunja** (zarządzanie zadaniami)
    *   **Kali Linux** (środowisko do testów bezpieczeństwa)
*   **DMZ** (`10.30.0.0/24` | `vmbr2`) – strefa zdemilitaryzowana izolująca usługi wystawione na zewnątrz:
    *   **Nginx Proxy Manager** (reverse proxy, zarządzanie ruchem przychodzącym)
    *   **VaultWarden** (menedżer haseł)
    *   **Minecraft Server** (serwer gry)
*   **Kurs** (`10.40.0.0/24` | `vmbr3`) – wyizolowane środowisko testowe Windows (lab **Active Directory**):
    *   **Windows Server 2022** (Nody: London, Glasgow, Manchester, Core)
    *   Stacje klienckie (**Windows 10** - CL1, CL2)

### Hardware Specifications
Host **Proxmox VE** działa na poniższej konfiguracji sprzętowej:
*   **CPU**: Intel Core i3-7100 @ 3.90GHz (2 rdzenie)
*   **RAM**: ~24 GB (23.36 GiB)
*   **Storage (Root)**: 40 GB 
*   **Storage (additional)**: 500 GB
*   **OS/Kernel**: Proxmox VE (Linux 7.0.0-3-pve)

### Security & Firewall Rules (DMZ)
Strefa **DMZ** posiada ścisłą izolację od reszty infrastruktury, aby zapobiec potencjalnemu atakowi typu **pivoting**. Główne reguły (od góry do dołu) na interfejsie **DMZ** w **pfSense**:

*   **Block** – ruch **TCP/UDP** z **DMZ** do podsieci **LAN**.
*   **Block** – ruch **TCP/UDP** z **DMZ** do podsieci **VPN** (WireGuard).
*   **Block** – ruch **ICMP** z **DMZ** do **VPN** oraz **LAN**.
*   **Pass** – ruch **ICMP** z **DMZ** do dowolnego miejsca (dzięki powyższym regułom blokującym, ping z DMZ wychodzi wyłącznie na **WAN**).
*   **Pass** – ruch **DNS** (UDP port 53) w obrębie podsieci **DMZ**.
