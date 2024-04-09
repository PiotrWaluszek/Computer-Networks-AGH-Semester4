
# Zadanie A: Rutowanie dynamiczne OSPF

## 1. Połączenie Routerów
Podłącz dwa routery Cisco przy użyciu kabli Ethernet lub Serial. Skonfiguruj interfejsy `loopback` na każdym z routerów.

## 2. Diagnostyka Sieci (Opcjonalnie)
Możesz podłączyć dodatkowe interfejsy do stacji PC dla celów diagnostycznych (ping).

- Użyj polecenia ping z interfejsu loopback:
  ```
  Router#ping 200.200.200.1 source 100.100.100.1
  ```
- Sprawdź, czy adres źródłowy ICMP oraz adres docelowy są poprawnie skonfigurowane.

## 3. Włączenie Routingu IP
- Przejdź do trybu konfiguracji routera:
  ```
  Router(config)#
  ```
- Włącz routing IP:
  ```
  Router(config)#ip routing
  ```
- Ustaw routing jako classless (bezklasowy).

## 4. Uruchomienie Routingu OSPF w Pojedynczym Obszarze (Area)

### Konfiguracja OSPF
- Uruchom routing OSPF z identyfikatorem procesu 150:
  ```
  Router(config)#router ospf 150
  ```

### Rejestracja Sieci
- Zarejestruj sieci bezpośrednio podłączone do domeny OSPF:
  ```
  Router(config-router)#network 200.200.200.0 0.0.0.255 area 0
  Router(config-router)#network 200.200.201.0 0.0.0.255 area 0
  ```

### Weryfikacja Stanu OSPF
- Sprawdź stan OSPF i interfejsy:
  ```
  Router#show ip protocols
  Router#show ip interface brief
  Router#show ip ospf interface fa 0/0
  ```

### Konfiguracja Sąsiedztwa OSPF
- W przypadku sieci NBMA zarejestruj sąsiadów OSPF:
  ```
  Router(config-router)#neighbor 200.200.100.1
  ```

### Weryfikacja Sąsiedztwa OSPF
- Sprawdź sąsiadów OSPF i stan protokołu routingu:
  ```
  Router#show ip ospf neighbor
  ```

### Ustawienia Priorytetu Interfejsu
- W przypadku sieci point-to-point lub point-to-multipoint, gdzie DR i BDR nie są wybierane, ustaw priorytet interfejsu na 0:
  ```
  Router(config-if)#ip ospf priority 0
  ```

### Weryfikacja Ustawień Interfejsu OSPF
- Sprawdź konfigurację interfejsu OSPF:
  ```
  Router#show ip ospf interface
  ```

## 5. Konfiguracja Intervałów Hello i Dead
- Zmiana interwałów hello i dead time dla różnych typów sieci OSPF:
  ```
  Router(config-if)#ip ospf hello-interval 10
  Router(config-if)#ip ospf dead-interval 40
  ```

## 6. Konfiguracja Kosztów OSPF
- Eksperymentalne zmienianie kosztu dla interfejsów OSPF:
  ```
  Router(config-if)#ip ospf cost <wartość>
  ```

## 7. Debugowanie OSPF
- Polecenia do debugowania i diagnostyki OSPF:
  ```
  Router#debug ip ospf adjacency
  Router#debug ip ospf events
  Router#show ip route
  Router#show ip ospf interface
  ```

## 8. Segmentacja i Logika OSPF
- Wymogi dla topologii logicznej OSPF i konfiguracja wirtualnych linków:
  ```
  RouterABR1(config)#router ospf 150
  RouterABR1(config-router)#area 1 virtual-link 5.5.5.5
  RouterABR2(config)#router ospf 150
  RouterABR2(config-router)#area 1 virtual-link 6.6.6.6
  ```

## 9. Konfiguracja Router ID
- Pamiętaj, że 5.5.5.5 i 6.6.6.6 to router ID, a nie adresy IP. Konfiguracja router ID:
  ```
  Router(config)#router ospf 150
  Router(config-router)#router-id 5.5.5.5
  ```

## 10. Tworzenie i Weryfikacja Virtual Links
- Tworzenie OSPF virtual link między wybranymi ABR:
  ```
  RouterABR1(config-router)#area 1 virtual-link 6.6.6.6
  RouterABR2(config-router)#area 1 virtual-link 5.5.5.5
  ```
- Weryfikacja virtual links:
  ```
  RouterABR1#show ip ospf virtual-links
  ```

## 11. Dodawanie Nowego Obszaru OSPF
- Aby utworzyć nowy obszar OSPF, dodaj nowe sieci do procesu OSPF i określ identyfikator area:
  ```
  Router(config)#interface loopback 5
  Router(config-if)#ip addr 200.200.101.1 255.255.255.0
  Router(config-if)#exit
  Router(config)#router ospf 150
  Router(config-router)#network 200.200.101.0 0.0.0.255 area 1
  ```
- Sprawdź zawartość tablicy routingu dla nowego obszaru:
  ```
  Router#show ip route
  ```
- Weryfikacja funkcjonowania OSPF w nowym obszarze:
  ```
  Router#show ip ospf border-routers
  Router#show ip ospf virtual-links
  ```


# Zadanie B: Redystrybucja Tras Pomiędzy Protokołami IGP

1. Redystrybucja tras pozwala na wymianę informacji o trasach pomiędzy różnymi protokołami routingu wewnątrz (IGP) i między systemami autonomicznymi (EGP).

2. Budowa instalacji z routerów R1, R2, R3 oraz stacji PC i odpowiednia konfiguracja interfejsów loopback.

3. Nadanie adresacji IP zgodnie z ogólnymi zasadami i ustawienie protokołów routingu:
   - R1: tylko RIP.
   - R2: RIP i OSPF.
   - R3: tylko OSPF.

4. Włączenie odpowiednich protokołów routingu dynamicznego na routerach i rejestracja sieci.

5. Weryfikacja zawartości tabeli routingu na routerach R1 i R3.

6. Konfiguracja redystrybucji z OSPF do RIP na routerze R2:
   ```
   R2(config)#router ospf 150
   R2(config-router)#redistribute rip metric 170 subnets
   ```
   - Weryfikacja, czy informacje o trasach z RIP zostały poprawnie oznaczone jako OSPF External.

7. Konfiguracja redystrybucji z RIP do OSPF na routerze R2:
   ```
   R2(config)#router rip
   R2(config-router)#redistribute ospf 150 metric 11
   ```
   - Weryfikacja, czy informacje o trasach z OSPF dotarły do routera R1.

Uwagi:
- Redystrybucja z RIP do OSPF wymaga użycia słowa kluczowego `subnets`.
- Informacje o sieciach klasyful będą propagowane jako takie, chyba że sieć ma maskę z klasy literowej adresacji IP (nie dotyczy VLSM).

- Uwaga dotycząca redystrybucji do RIP: Konwersja metryki trasy jest kończona, a trasy RIP są oznaczone liczbą przeskoków z maksymalną długością trasy równą 15.

8. Redystrybucja RIP <-> EIGRP
Usuń z routerów OSPF i uruchom w jego miejscu EIGRP. Następnie skonfiguruj redystrybucję tras między RIP i EIGRP i przetestuj ten mechanizm.


# Zadanie C: Tunele GRE Odległych Sieci Wielosegmentowych IP

1. Stworzenie spójnego rozproszonego systemu routowania dynamicznego przez Internet za pomocą tunelowania GRE.

2. Konfiguracja adresacji interfejsów fizycznych routerów R1, R2, R3:
   ```
   R1(config)#interface FastEthernet0/0
   R1(config-if)#ip address 200.200.200.1 255.255.255.0
   R2(config)#interface FastEthernet0/0
   R2(config-if)#ip address 200.200.200.2 255.255.255.0
   R2(config)#interface FastEthernet0/1
   R2(config-if)#ip address 200.200.201.2 255.255.255.0
   R3(config)#interface FastEthernet0/0
   R3(config-if)#ip address 200.200.201.1 255.255.255.0
   ```

3. Konfiguracja tunelu GRE między R1 i R3 przez R2:
   - Na R1:
     ```
     R1(config)#interface Tunnel0
     R1(config-if)#ip address 10.0.0.1 255.255.255.0
     R1(config-if)#tunnel source FastEthernet0/0
     R1(config-if)#tunnel destination 200.200.201.1
     ```
   - Na R3:
     ```
     R3(config)#interface Tunnel0
     R3(config-if)#ip address 10.0.0.2 255.255.255.0
     R3(config-if)#tunnel source FastEthernet0/0
     R3(config-if)#tunnel destination 200.200.200.1
     ```

3. Mechanizm routowania między R1 i R3 z wykorzystaniem tuneli GRE:
   ```
   R1(config)#ip route 200.200.201.0 255.255.255.0 200.200.200.2
   R3(config)#ip route 200.200.200.0 255.255.255.0 200.200.201.2
   ```

4. Definicja tuneli GRE na routerach R1 i R3:
   - Na R1:
     ```
     R1(config)#interface Tunnel0
     R1(config-if)#tunnel source FastEthernet0/0
     R1(config-if)#tunnel destination 200.200.201.1
     R1(config-if)#ip address 192.168.5.1 255.255.255.0
     ```
   - Na R3:
     ```
     R3(config)#interface Tunnel0
     R3(config-if)#tunnel source FastEthernet0/0
     R3(config-if)#tunnel destination 200.200.200.1
     R3(config-if)#ip address 192.168.5.2 255.255.255.0
     ```
   - Sprawdzenie działania tunelu GRE:
     ```
     R1#ping 192.168.5.2
     R1#show interface tunnel 0
     ```

5. Konfiguracja interfejsów loopback i OSPF na R1 i R3:
   ```
   R1(config)#interface Loopback0
   R1(config-if)#ip address 192.168.0.1 255.255.255.0
   R1(config)#router ospf 1
   R1(config-router)#network 192.168.0.0 0.0.0.255 area 0
   R1(config-router)#network 192.168.5.0 0.0.0.255 area 0
   R3(config)#interface Loopback0
   R3(config-if)#ip address 192.168.1.1 255.255.255.0
   R3(config)#router ospf 1
   R3(config-router)#network 192.168.1.0 0.0.0.255 area 0
   R3(config-router)#network 192.168.5.0 0.0.0.255 area 0
   ```

6. OSPF diagnostyka i sprawdzenie stanu sąsiedztwa:
   ```
   R1(config-router)#log-adjacency-changes
   R1#show ip ospf neighbor
   ```

7. Testowanie funkcjonalności routowania poprzez tunel GRE:
   - Polecenia do sprawdzenia trasy przez tunel:
     ```
     R1#ping <adres_IP_z_sieci_R3>
     R1#traceroute <adres_IP_z_sieci_R3>
     R1#show ip route
     ```
   - Sprawdzenie czy trasa jest prowadzona przez tunel GRE, a nie przez łącze fizyczne.

8. Konfiguracja MTU i MSS dla tunelu GRE:
   ```
   R1(config)#interface Tunnel0
   R1(config-if)#ip mtu 1460
   R1(config-if)#ip tcp adjust-mss 1430
   R3(config)#interface Tunnel0
   R3(config-if)#ip mtu 1460
   R3(config-if)#ip tcp adjust-mss 1430
   ```
   - Ustawienie MTU (Maximum Transmission Unit) i MSS (maximum segment size) zapewnia, że transmisja w tunelu nie będzie miała fragmentacji końcowej.
   - Należy ustawić identyczne wartości MTU i MSS na obu końcach tunelu.

