Naravno. Do ovog trenutka smo završili **osnovnu konfiguraciju P1 i P2 + OSPF između njih**. Ovo je dobar trenutak za rezime, jer od sada krećemo prema PE ruterima i kasnije MPLS L3VPN delu.

# REZIME DOSADAŠNJEG LABA

Naša trenutna topologija je:

```text
              15.15.15.0/24          12.12.12.0/24
        PE1 ---------------- P1 ---------------- P2
                             
                              26.26.26.0/24
                                      |
                                     PE2
```

Za sada su **P1 i P2 potpuno podešeni u core delu**.

---

# 1. P1 — osnovna konfiguracija

P1 ima:

```text
Loopback0       1.1.1.1/32
Gi0/0           15.15.15.1/24
Gi0/1           12.12.12.1/24
```

Konfiguracija koju smo uneli:

```cisco
enable
configure terminal

ip cef
mpls ip
mpls label protocol ldp

interface Loopback0
 ip address 1.1.1.1 255.255.255.255
 exit

interface GigabitEthernet0/0
 ip address 15.15.15.1 255.255.255.0
 no shutdown
 mpls ip
 exit

interface GigabitEthernet0/1
 ip address 12.12.12.1 255.255.255.0
 no shutdown
 mpls ip
 exit

end
```

### Šta smo ovde uradili?

**`ip cef`**

Uključili smo Cisco Express Forwarding, koji je osnova za efikasno prosleđivanje paketa.

**`mpls ip`**

Uključili smo MPLS funkcionalnost na ruteru.

**`mpls label protocol ldp`**

Definisali smo **LDP** kao protokol za distribuciju MPLS labela.

**Loopback0**

```cisco
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
```

P1 dobija stabilan `/32` identitet.

**Gi0/0**

```text
15.15.15.1/24
```

Veza:

```text
PE1 ↔ P1
```

**Gi0/1**

```text
12.12.12.1/24
```

Veza:

```text
P1 ↔ P2
```

Na oba core interfejsa uključili smo:

```cisco
mpls ip
```

---

# 2. P1 — OSPF

Zatim smo na P1 uključili OSPF:

```cisco
configure terminal

router ospf 1
 router-id 1.1.1.1
 network 1.1.1.1 0.0.0.0 area 0
 network 12.12.12.0 0.0.0.255 area 0
 network 15.15.15.0 0.0.0.255 area 0
 exit

end
```

### Šta ovo znači?

Napravili smo:

```text
OSPF process 1
Router-ID = 1.1.1.1
Area = 0
```

I rekli P1 da kroz OSPF učestvuju:

```text
1.1.1.1/32
12.12.12.0/24
15.15.15.0/24
```

Dakle P1 sada može da razmenjuje routing informacije sa drugim OSPF ruterima u provider core-u.

---

# 3. P2 — osnovna konfiguracija

P2 ima:

```text
Loopback0       2.2.2.2/32
Gi0/0           12.12.12.2/24
Gi0/1           26.26.26.2/24
```

Konfiguracija:

```cisco
enable
configure terminal

ip cef
mpls ip
mpls label protocol ldp

interface Loopback0
 ip address 2.2.2.2 255.255.255.255
 exit

interface GigabitEthernet0/0
 ip address 12.12.12.2 255.255.255.0
 no shutdown
 mpls ip
 exit

interface GigabitEthernet0/1
 ip address 26.26.26.2 255.255.255.0
 no shutdown
 mpls ip
 exit

end
```

Dakle:

```text
P1 Gi0/1       12.12.12.1
       |
       |
P2 Gi0/0       12.12.12.2
```

i:

```text
P2 Gi0/1       26.26.26.2
       |
       |
      PE2
```

---

# 4. P2 — OSPF

Na kraju smo P2 uključili u isti OSPF proces:

```cisco
configure terminal

router ospf 1
 router-id 2.2.2.2
 network 2.2.2.2 0.0.0.0 area 0
 network 12.12.12.0 0.0.0.255 area 0
 network 26.26.26.0 0.0.0.255 area 0
 exit

end
```

Dakle P2 sada ima:

```text
OSPF process 1
Router-ID = 2.2.2.2
Area 0
```

---

# 5. NAJVAŽNIJA PROVERA — P1 ↔ P2

Na P2 smo dobili:

```text
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   FULL/DR         00:00:31    12.12.12.1      GigabitEthernet0/0
```

Ovo je **odličan rezultat**.

Posebno:

```text
1.1.1.1
```

→ P1 OSPF Router-ID.

```text
FULL/DR
```

→ OSPF susedstvo je uspešno formirano.

```text
12.12.12.1
```

→ P1 adresa na zajedničkom linku.

```text
GigabitEthernet0/0
```

→ P2 interfejs preko kojeg je sused pronađen.

Dakle imamo:

```text
                    OSPF AREA 0

        12.12.12.0/24
P1 =========================== P2
1.1.1.1                       2.2.2.2
    FULL OSPF NEIGHBORSHIP
```

## Trenutno stanje

| Uređaj | Osnovni IP | CEF | MPLS | LDP protokol | OSPF | Sused |
| ------ | ---------- | --- | ---- | ------------ | ---- | ----- |
| P1     | ✅          | ✅   | ✅    | LDP          | ✅    | P2    |
| P2     | ✅          | ✅   | ✅    | LDP          | ✅    | P1    |

**Dakle, do ovog trenutka sve je ispravno.** Ne treba ništa menjati niti popravljati.

Jedina stvar koju **još nismo uradili**, i koja je navedena u originalnom zadatku, jeste aktiviranje/konfigurisanje **LDP router-ID/autoconfig**. To ćemo raditi u odgovarajućem sledećem koraku, zajedno sa proverom MPLS/LDP susedstva.

Za sada je naš core IP/OSPF deo **P1 ↔ P2 funkcionalan**.
