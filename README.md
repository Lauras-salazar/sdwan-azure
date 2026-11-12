# SD-WAN en Azure: orquestador (hub)

Automatiza la red overlay WireGuard con Ansible. El hub es el centro (10.10.0.1) y cada sede es un peer.

## Archivos
- `inventario.ini`: lista de VMs con IP pública, usuario, IP de túnel y red local.
- `playbook.yml`: instala paquetes, genera llaves y levanta `wg0` en todas las VMs.
- `wg0.conf.j2`: plantilla Jinja2 de `wg0.conf`. El hub recibe un `[Peer]` por sede; cada sede apunta al hub.
- `ansible.cfg`: configuración mínima.

## Cómo agregar una sede nueva (zero-touch)
1. Crear la VM (Ubuntu, TCP 22 y UDP 51820 abiertos, IP forwarding activo).
2. Agregar una línea al grupo `[sedes]` de `inventario.ini`:
   `nombre ansible_host=IP ansible_user=usuario tunnel_ip=10.10.0.X lan=192.168.Y.0/24`
   (`lan` es opcional si la sede no tiene red local).
3. Correr: `ansible-playbook playbook.yml`

El playbook genera las llaves de la sede nueva y reescribe el `wg0.conf` del hub con el nuevo peer.
Nota: reescribe el `wg0.conf` de todas las VMs del inventario (guarda copia de respaldo con `backup: true`).

## Verificar
- `sudo wg show` (debe aparecer `latest handshake` en cada peer)
- `ping 10.10.0.2`, `ping 10.10.0.3`, `ping 10.10.0.4`

## Direccionamiento
| Nodo  | Túnel wg0 | Red local |
|-------|-----------|-----------|
| Hub   | 10.10.0.1 | -         |
| Oscar | 10.10.0.2 | 192.168.10.0/24 |
| Luis  | 10.10.0.3 | 192.168.20.0/24 |
| Juan  | 10.10.0.4 | -         |
