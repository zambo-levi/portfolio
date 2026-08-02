# Portfolio Technique — ZAMBO LEVI
## Licence Professionnelle ASUR (Administration & Sécurité des Réseaux)

Bienvenue sur mon portfolio professionnel. Vous trouverez ci-dessous la documentation technique de mes réalisations majeures.

---

## 🛡️ Étude de Cas : Sécurisation d'un réseau VLAN par l'intégration d'un pare-feu Next-Gen FortiGate

### 📋 1. Présentation du Projet & Cahier des Charges
Ce projet formalise la conception et la sécurisation d'une infrastructure réseau segmentée et hautement cloisonnée, simulée au sein de l'environnement **PNETLab**. L'objectif est d'appliquer une politique stricte de type *Least Privilege* en utilisant un pare-feu FortiGate (VM64-KVM) comme passerelle de routage inter-VLAN et de filtrage applicatif.

---

### 🗺️ 2. Architecture Réseau & Topologie
La topologie s'articule autour d'un routeur/switch central distribuant quatre zones réseau distinctes vers les interfaces du pare-feu FortiGate :

*   **VLAN 10 (Administration) :** Subnet `192.168.10.0/24` — Zone critique contenant le poste d'administration réseau.
*   **VLAN 20 (Utilisateurs) :** Subnet `192.168.20.0/24` — Zone des collaborateurs internes.
*   **VLAN 30 (Invités) :** Subnet `192.168.30.0/24` — Zone visiteurs isolée.
*   **VLAN 99 (Management) :** Subnet `192.168.99.0/24` — Zone d'administration des infrastructures.
*   **Interface WAN (port2) :** Interconnexion vers le réseau externe/Internet.

![Topologie Réseau PNETLab](./images/Topologie.png)

---

### ⚙️ 3. Implémentation des Règles de Filtrage (Firewall Policy)
Les politiques de flux implémentées sur le FortiGate traduisent les exigences strictes de sécurité de la maquette :

1.  **Gestion des accès Admin (VLAN 10) :** Autorisation complète vers les zones Utilisateurs (VLAN 20), Administration (VLAN 10), et Management (VLAN 99).
2.  **Accès Internet Privilégié (VLAN 10 & VLAN 20/30) :** Activation du NAT (Enabled) pour les sorties WAN avec profils de filtrage dédiés.
3.  **Règle de Blocage Implicite (Implicit Deny) :** Tout flux inter-zone non explicitement listé est immédiatement rejeté (Action: `DENY`).

![Règles de Pare-feu FortiGate](./images/R%C3%A8gles%20de%20pare%20feu.png)

---

### 🎯 4. Phase de Recette & Validation (Preuves Concrètes)

La validation de la politique de sécurité a été testée et validée via des captures de terminaux VPCS et l'analyse en temps réel du journal des flux du FortiGate.

#### A. Test de Connectivité WAN (Succès attendu)
Depuis la zone Invité, le trafic à destination d'Internet (IP de test `8.8.8.8`) est pleinement fonctionnel. Les logs internes confirment le passage avec succès via la règle ID 3 (Résultat : `84 B / 84 B`).

![Ping Invités Internet](./images/Invit%C3%A9s%20vers%20internet%20autoris%C3%A9s.png)

#### B. Test de Cloisonnement Inter-VLAN (Blocage attendu)
Une tentative de communication ICMP initiée depuis le poste **Utilisateur** (`192.168.20.10`) vers la zone sensible **Administration** (`192.168.10.10`) se solde par un échec systématique (`timeout`). 

Le module *Forward Traffic* du FortiGate intercepte et journalise immédiatement ces violations via l'Action `Deny: policy...` sous la règle implicite (Policy ID 5).

![Blocage Utilisateur vers Admin](./images/utilisa%20vers%20admin.png)
![Logs des Blocages FortiGate](./images/Logs%20pare%20feu.png)

#### C. Validation de la Passerelle de Management
Le bon fonctionnement des sous-interfaces et de la passerelle du FortiGate est validé par un diagnostic réseau réussi depuis l'hôte de Management vers son interface d'infrastructure dédiée en `192.168.99.1`.

![Ping Management Passerelle](./images/Ping%20pare%20feu%20depuis%20vlan%2099.png)
