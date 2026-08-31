# aerobot-tunnel-server (POC)

Túnel [frp](https://github.com/fatedier/frp) dockerizado.
`frpc` corre en el robot y se conecta de salida al `frps` del EC2.

```
robot 127.0.0.1:8000  ──frpc──►  EC2:7000        (control)
                                 EC2:8080  ◄────  internet
```

## 1. Security group del EC2

Abre en el SG de tu instancia (consola AWS o tu Terraform):

| Puerto | Origen        | Para qué                 |
|--------|---------------|--------------------------|
| 7000   | 0.0.0.0/0     | conexión de control frpc |
| 8080   | 0.0.0.0/0     | servicio del robot       |

## 2. En el EC2

```bash
# instalar docker (Amazon Linux 2023)
sudo dnf install -y docker && sudo systemctl enable --now docker

# subir la carpeta frps/ y levantar
cd frps
# edita frps.toml: cambia el token
sudo docker compose up -d
sudo docker compose logs -f
```

## 3. En el robot

```bash
cd frpc
# edita frpc.toml: pon la IP del EC2, el MISMO token, y localPort real
sudo docker compose up -d
sudo docker compose logs -f      # debe decir "start proxy success"
```

## 4. Probar

```bash
curl http://IP_DEL_EC2:8080/
```

WebSocket pasa transparente (es TCP crudo). El tráfico en `:8080` va en HTTP
plano — suficiente para POC.
