# Relatório de Projeto Final A e B
## Tema: Cloud Computing e Virtualization
### IPT - Instituto Politécnico de Tomar
### Por: José Ngoma Júnior | Aluno número: 25162

## Introdução

O projeto visa criar uma infraestrutura de balanceamento de carga em dois ambientes:
- **Projeto A**: Utilizando Vagrant.
- **Projeto B**: Utilizando Docker.
Ambos os projetos utilizam Nginx para distribuir solicitações entre servidores PHP, garantindo carga balanceada, alta disponibilidade e escalabilidade.

## Problema a ser Resolvido

A alta demanda de solicitações pode sobrecarregar um único servidor, resultando em lentidão ou indisponibilidade. O balanceamento de carga distribui solicitações entre vários servidores, melhorando a disponibilidade e performance do sistema.

## Solução Proposta

### Projeto A

- **Componentes**: Dois servidores PHP e um balanceador de carga Nginx.
- **Provisionamento**: Uso do Vagrant para configurar máquinas virtuais.
- **Escalabilidade**:
  - Vertical: Adicionar mais RAM aos servidores.
  - Horizontal: Usar Nginx para balanceamento de carga entre dois servidores.

#### Passos para Configuração

1. Acessar o servidor com `vagrant ssh`.
2. Inicializar o Hypervisor com `vagrant up`.
3. Criar arquivo de configuração: `sudo touch /etc/nginx/conf.d/load-balancer.conf`.
4. Remover bloqueio padrão: `sudo rm -r /etc/nginx/sites-enabled/default`.
5. Reiniciar Nginx: `sudo /etc/init.d/nginx restart`.
6. Modificar configuração:
    ```nginx
    upstream nodes {
        server 192.168.44.11;
        server 192.168.44.12;
    }

    server {
        listen 80;
        location / {
            proxy_pass http://nodes;
        }
    }
    ```
7. Salvar e reiniciar Nginx: `sudo /etc/init.d/nginx restart`.

### Projeto B

Foco em escalabilidade e alta disponibilidade usando Docker e Azure Container Registry.

#### Componentes

- Dois servidores PHP (`projectb1` e `projectb2`).
- Dois balanceadores de carga Nginx (`load_balancer1` e `load_balancer2`).
- Um balanceador de carga principal Nginx (`main_load_balancer`).

#### Configuração e Implementação

##### Docker Compose

Arquivo `docker-compose.yml`:

```yaml
version: '3.9'

services:
  projectb1:
    image: projectb1
    container_name: projectb11
    ports:
      - "8081:80"
    networks:
      - webnet

  projectb2:
    image: projectb2
    container_name: projectb22
    ports:
      - "8082:80"
    networks:
      - webnet

  load_balancer1:
    image: nginx:latest
    volumes:
      - ./nginx/load_balancer1.conf:/etc/nginx/nginx.conf
    ports:
      - "8084:80"
    depends_on:
      - projectb1
      - projectb2
    networks:
      - webnet

  load_balancer2:
    image: nginx:latest
    volumes:
      - ./nginx/load_balancer2.conf:/etc/nginx/nginx.conf
    ports:
      - "8083:80"
    depends_on:
      - projectb1
      - projectb2
    networks:
      - webnet

  main_load_balancer:
    image: nginx:latest
    volumes:
      - ./nginx/main_load_balancer.conf:/etc/nginx/nginx.conf
    ports:
      - "80:80"
    depends_on:
      - load_balancer1
      - load_balancer2
    networks:
      - webnet

networks:
  webnet:
```

##### Configuração do Nginx

Arquivos de configuração no diretório `nginx`.

**nginx/main_load_balancer.conf**:

```nginx
events {
    worker_connections 1024;
}

http {
    upstream load_balancers {
        server load_balancer1:80 max_fails=3 fail_timeout=30s;
        server load_balancer2:80 max_fails=3 fail_timeout=30s;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://load_balancers;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

**nginx/load_balancer1.conf** e **nginx/load_balancer2.conf**:

```nginx
events {
    worker_connections 1024;
}

http {
    upstream php_servers {
        server projectb1:80;
        server projectb2:80;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://php_servers;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

## Benefícios da Solução

- **Distribuição de Carga**: Evita sobrecarga de um único servidor.
- **Alta Disponibilidade**: Continuidade do sistema mesmo se um servidor falhar.
- **Escalabilidade**: Adição fácil de novos servidores e balanceadores de carga.

## Possíveis Melhorias

### Escalabilidade

- **Auto-scaling**: Utilizar Kubernetes.
- **Cloud-based Load Balancing**: Balanceadores de carga na nuvem.

### Alta Disponibilidade

- **Redundância Geográfica**: Servidores em múltiplas regiões.
- **Database Replication**: Garantir disponibilidade dos dados.

### Performance

- **Cache de Conteúdo**: Reduzir carga nos servidores PHP.
- **Otimização de Código**: Melhorar eficiência do código PHP e consultas ao banco de dados.

## Conclusão

O projeto demonstrou a implementação de soluções de balanceamento de carga utilizando Vagrant no Projeto A e Docker no Projeto B, ambos com Nginx. As melhorias sugeridas podem tornar o sistema mais robusto e eficiente. Para produção, recomenda-se o uso de soluções mais avançadas e robustas de balanceamento de carga e gerenciamento de contêineres.
