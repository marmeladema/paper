% Internet Protocol version 6
% Élie ROUDNINSKI
% 19/11/2017

# Organisation

Pour effectuer les éxercies, vous pouver travailler :

1. en binôme avec chacun un poste connecté ensemble par un câble Ethernet
2. seul avec des machines virtuelles connectées à une interface virtuelle

Dans ce document, le premier membre du binôme sera appelé arbitrairement Michel·le (M1) et le second Michel·le-Michel·le (M2).

Il est conseillé que chacun des membres effectue les exercices.

# Pré-requis

La distribution Linux utilisée est laissé libre de choix mais les outils suivant devront être disponible (ou équivalents) :

- « `ip` » (paquet « `iproute2` » sous Linux/Debian)
- « `wireshark` » 
- « `scapy` » 
- « `radvd` »
- « `miredo` »

Certaines commandes peuvent nécessiter des droits administrateurs.

# Astuces

Pour ceux qui utilisent NetworkManager, il est conseillé de désactiver la gestion de l’interface par celui-ci :

## /etc/NetworkManager/NetworkManager.conf
<div lang="en-US">
```{.txt .numberLines}
[keyfile]
unmanaged-devices=interface-name:<iface>
```
</div>

# 1. SLAAC

\small

## 1.1. M1 devra désactiver IPv6 pour son interface :

. . .

`$ echo 1 > /proc/sys/net/ipv6/conf/<iface>/disable_ipv6`

## 1.2. M2 devra désactiver son interface :

. . .

`$ ip link set <iface> down`

## 1.3. M1 devra capturer le trafic IPv6 de son interface :

. . .

`$ wireshark -i <iface> -k -f ip6`

## 1.4. M2 devra activer son interface :

. . .

`$ ip link set <iface> up`

# 1. SLAAC

(@) Quels sont les paquets envoyés ?

(@) Quel est la structure du paquet « *Neighbor Solicitation* » ?

| On pourra recommencer en utilisant d'autres systèmes d'exploitation pour M2 (Windows, Android, BSD, etc).

# 1. SLAAC

M1 va maintenant utiliser **Scapy** pour envoyer un message *Router Advertisement* à M2.

. . .

\small

<div lang="en-US">

```python
>> p = Ether()/
       IPv6(src="fe80::", dst="ff02::1")/
       ICMPv6ND_RA()/
       ICMPv6NDOptPrefixInfo(prefix="2001:db8::", prefixlen=64)
>> sendp(p, iface="tap0")
```

| Remplacez l'adresse `src` par votre adresse de lien local en `fe80::/64`.

</div>

# 1. SLAAC

<div lang="en-US">

(@) À quoi correspond l'adresse `ff02::1` ?

(@) Quels sont les paquets envoyés ?

(@) Listez vos adresses et vos routes, que voyez vous ?

| On pourra recommencer en utilisant d'autres systèmes d'exploitation pour M2 (Windows, Android, BSD, etc).

</div>

# 1. SLAAC

## M1 devra ré-activer IPv6 pour son interface

. . .

`$ echo 0 > /proc/sys/net/ipv6/conf/<iface>/disable_ipv6`

## M1 devra ajouter une adresse dans le préfixe annoncé

. . .

<div lang="en-US">
`$ ip -6 addr add 2001:db8::1/64`
</div>

## M2 devra vérifier qu'il peut contacter M1

. . .

<div lang="en-US">
`$ ping6 2001:db8::1`
</div>

# 1. SLAAC

M1 va maintenant utiliser Scapy pour envoyer un message *Router Advertisement* à M2.

. . .

\small

<div lang="en-US">

```python
>> p = Ether()/
       IPv6(src="2001:db8::1", dst="ff02::1")/
       ICMPv6ND_RA()/
       ICMPv6NDOptPrefixInfo(prefix="2002:db8::", prefixlen=64)
>> sendp(p, iface="tap0")
```

| Remplacez l'adresse `src` par votre **nouvelle** adresse et le `prefix` par un **nouveau** préfixe.

</div>

# 1. SLAAC

<div lang="en-US">

(@) Que remarquez-vous ?

(@) Pourquoi ce comportement selon-vous ?

</div>

# 1. SLAAC DoS

M2 devra désactiver son interace.

M1 devra maintenant utiliser THC-IPV6 pour effectuer un déni de service sur M2 grâce à l'outil `dos-new-ip6`.

M2 devra activer son interface.

M2 devra essayer de ping l'interface

Recommencez en modifiant la valeur dans :

`/proc/sys/net/ipv6/conf/<iface>/dad_transmits`

# 1. SLAAC DoS

(@) Listez les adresses. Que constatez-vous ?

(@) Comment fonctionne cette « attaque » ?

(@) Quelle influence a la valeur `dad_transmits`

# 1. SLAAC + radvd

M1 devra configurer `radvd`

<div lang="en-US">
```{.numberLines}
interface <iface> {
	AdvSendAdvert on;
};
```
</div>

Puis recommencer la toute première étape du TP en activant IPv6 sur M1.

| N'oubliez pas redémarrer `radvd`

# 1. SLAAC + radvd

(@) Quelles sont les différences par rapport au SLAAC de la première étape ?

# 1. SLAAC + radvd

M1 devra configurer `radvd`

<div lang="en-US">
```{.numberLines}
interface <iface> {
	AdvSendAdvert on;
	prefix 2001:db8::/64 {
	};
};
```
</div>

Puis recommencer l'étape précédante.

| N'oubliez pas redémarrer `radvd`

# 1. SLAAC + radvd

(@) Quelles sont les différences par rapport au SLAAC de l'étape précédante ?

# 1. SLAAC avec adresses temporaires

Recommencez l'étape précédante en ayant activé les adresses temporaires.

`$ echo 2 > /proc/sys/net/ipv6/conf/<iface>/use_tempaddr`

(@) Que remarquez-vous ?

# 1. SLAAC avec adresses stables

Recommencez l'étape précédante en ayant activé les adresses stables.

<div lang="en-US">

\small

`$ echo 'AA:..:XX' > /proc/sys/net/ipv6/cong/<iface>/stable_secret`

</div>

Sur certaines versions de Linux, il est nécéssaire d'activer les adresses stables avec ip :

`ip link set <iface> addrgenmode stable_secret`

(@) Que remarquez-vous ?

# 1. SLAAC + radvd + RDNSS

<div lang="en-US">
```{.numberLines}
interface <iface> {
	AdvSendAdvert on;
	prefix 2001:db8::/64 {
		AdvOnLink on;
		AdvAutonomous on;
		AdvRouterAddr on;
	};
	RDNSS 2001:db8::1 {
		AdvRDNSSLifetime 60;
	};
};
```
</div>

# 2. Tunnel 6in4

\small

## M* devra désactiver IPv6

<div lang="en-US">
`$ echo 1 > /proc/sys/net/ipv6/conf/<iface>/disable_ipv6`
</div>

## M* devra attribuer une adresse IPv4 à son interface

<div lang="en-US">
`$ ip addr add 192.168.0.X/24 dev <iface>`
</div>

## M* devra vérifier qu'il peut contacter son pair par IPv4

<div lang="en-US">
`$ ping 192.168.0.X`
</div>

## M* devra créer l'interface du tunnel

<div lang="en-US">
`$ ip tunnel add $TUN mode sit local $LOCAL remote $REMOTE`
</div>

## M* devra activer l'interface du tunnel

<div lang="en-US">
`$ ip link set dev $TUN up`
</div>

# 2. Tunnel 6in4

M* devra vérfier qu'il peut contacter son pair par IPv6.

(@) Quel est la forme du trafic sur l'interface du tunnel ?

(@) Et sur l'interface physique ?

# 2. Tunnel 6in4 + radvd

## M1 devra configurer radvd pour qu'il gère l'interface

<div lang="en-US">
```
interface $TUN {
	AdvSendAdvert on;
	# sit interfaces do not support multicast
	UnicastOnly on;
	prefix 2001:db8::/64 {
	};
};
```
</div>

## M2 ré-activer son interface

<div lang="en-US">
`$ ip link set dev $TUN down`

`$ ip link set dev $TUN up`
</div>

# 2. Tunnel 6in4 + radvd

# 3. Miredo

# 3. Miredo + NAT

M1 :

<div lang="en-US">
```
interface <iface> {
	AdvSendAdvert on;
	prefix fec0::/64 {
	};
};
```
</div>

M1 :

`$ ip6tables -t nat -A POSTROUTING -o teredo -j MASQUERADE`

M1 :

`$ echo 1 > /proc/sys/net/ipv6/conf/all/forwarding`
