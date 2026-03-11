# Segurança de Rede com EVE-NG

<img width="940" height="467" alt="Topologia" src="https://github.com/user-attachments/assets/68fe9723-bc08-42af-8f21-d21f5ca72667" />

### Descrição

Implementação de uma topologia de rede segmentada em 3 VLANs, roteamento
inter-VLAN em um roteador Cisco com listas de acesso (ACLs) para aplicar políticas de
segurança, bloqueando tráfego não autorizado entre os segmentos e controlando
o acesso à internet. O projeto teve como objetivo **praticar os princípios de segmentação
de rede e controle de acesso de privilégio mínimo**.

---

### Topologia

Implementação de três VLANs com:

- 3 PCs (um em cada VLAN)  
- 2 Switches  
- 1 Roteador  
- 1 Ponte de Rede (Cloud)  

---

### Requisitos

- VMware Workstation Pro  
- EVE-NG (4 GB RAM, 2 CPUs, 20 GB HD, modo NAT)  
- Cisco IOL 3725  
- WinSCP  
- EVE-NG Integration Package  

---

### Estrutura de Rede

| VLAN | Sub-rede          | Host Exemplo        | Descrição            |
|------|-------------------|--------------------|----------------------|
| 10   | 192.168.10.0/24   | 192.168.10.10      | Rede interna 1       |
| 20   | 192.168.20.0/24   | 192.168.20.10      | Rede interna 2       |
| 30   | 192.168.30.0/24   | 192.168.30.10      | Rede administrativa  |

---

### ACLs (Controle de Acesso)

```bash
ip access-list extended VLAN-RESTRICOES
 ! VLAN 10 não acessa VLAN 20
 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255

 ! VLAN 20 não acessa VLAN 10 nem a internet
 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.20.0 0.0.0.255 any

 ! VLAN 30 tem acesso total
 permit ip 192.168.30.0 0.0.0.255 any

 ! Demais tráfegos são negados por padrão
 deny ip any any
```

---

## Config. do Roteador com ACL’s e NAT

```plaintext
enable
conf t

int fa1/0
ip address dhcp
no shut
exit

int fa0/0
no shut
exit

int fa0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
ip nat inside
exit

int fa2/0
no shut
exit

int fa2/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
ip nat inside
exit

int fa2/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
ip nat inside
exit

ip access-list standard NAT-VLANS
permit 192.168.10.0 0.0.0.255
permit 192.168.20.0 0.0.0.255
permit 192.168.30.0 0.0.0.255
exit

ip nat inside source list NAT-VLANS interface fa1/0 overload

ip access-list extended VLAN-RESTRICOES
deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
deny ip 192.168.20.0 0.0.0.255 any
permit ip 192.168.30.0 0.0.0.255 any
deny ip any any
exit

int fa0/0.10
ip access-group VLAN-RESTRICOES in
exit
int fa2/0.20
ip access-group VLAN-RESTRICOES in
exit

ip domain-lookup
ip name-server 8.8.8.8
ip name-server 1.1.1.1

end
wr
```


## Config. do Switch 1 com VLANs de Acesso

```plaintext
enable
conf t

vlan 10
name VLAN10
exit

int fa1/0
switchport mode access
switchport access vlan 10
no shut
exit

int fa1/15
switchport mode trunk
switchport trunk allowed vlan all
no shut
exit

end
wr
```


## Config. do Switch 2 com Trunk e VLANs

```plaintext
enable
conf t

vlan 20
name VLAN20
exit

vlan 30
name VLAN30
exit

int fa1/0
switchport mode access
switchport access vlan 20
no shut
exit

int fa1/1
switchport mode access
switchport access vlan 30
no shut
exit

int fa1/15
switchport mode trunk
switchport trunk allowed vlan all
no shut
exit

end
wr
```

## Config. do PC1 (VLAN 10)

```plaintext
enable
conf t
int fa0/0
ip address 192.168.10.10 255.255.255.0
no shut
exit
ip route 0.0.0.0 0.0.0.0 192.168.10.1
ip name-server 8.8.8.8
end
wr
```


## Config. do PC2 (VLAN 20)

```plaintext
enable
conf t
int fa0/0
ip address 192.168.20.10 255.255.255.0
no shut
exit
ip route 0.0.0.0 0.0.0.0 192.168.20.1
ip name-server 8.8.8.8
end
wr
```


## Config. do PC3 (VLAN 30)

```plaintext
enable
conf t
int fa0/0
ip address 192.168.30.10 255.255.255.0
no shut
exit
ip route 0.0.0.0 0.0.0.0 192.168.30.1
ip name-server 8.8.8.8
end
wr
```

## Resultado

O projeto mostrou, de forma prática, como utilizar VLANs, roteamento inter-VLAN e ACLs para segmentar uma rede e aplicar regras básicas de segurança. Cada VLAN representa um segmento separado, permitindo controlar quais redes podem ou não se comunicar entre si.

Com o uso de ACLs, foi possível bloquear o tráfego não autorizado entre as VLANs e permitir acesso apenas quando necessário. Já o NAT permitiu que as redes internas utilizassem a conexão externa, mantendo os endereços privados dentro da rede.

O projeto teve como objetivo principal praticar conceitos fundamentais de redes e segurança, como **segmentação, controle de acesso e organização de tráfego** em ambientes simulados utilizando o EVE-NG.
