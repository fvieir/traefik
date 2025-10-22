## Adicionar TLS e certificado ambiente de desenvolvimento
### Criar um certificado autoassinado

Referência: [traefik doc](https://doc.traefik.io/traefik/expose/docker/#enable-tls)

Gerar um certificado autoassinado:

```bash
mkdir -p certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout certs/local.key -out certs/local.crt \
  -subj "/CN=*.docker.localhost"
```

Crie um diretório para configuração dinâmica e adicione um arquivo de configuração TLS:
```bash
mkdir -p dynamic
cat > dynamic/tls.yml << EOF
tls:
  certificates:
    - certFile: /certs/local.crt
      keyFile: /certs/local.key
EOF
```

## Teste os Middlewares
Referência: [add middleweares](https://doc.traefik.io/traefik/expose/docker/#add-middlewares)

Teste o middleware Secure Headers:

```bash
curl -k -I -H "Host: whoami.docker.localhost" https://localhost/
```
Nos cabeçalhos de resposta, você deve ver os cabeçalhos de segurança definidos pelo middleware:
- X-Frame-Options: DENY
- X-Content-Type-Options: nosniff
- X-XSS-Protection: 1; mode=block
- Strict-Transport-Securitycom as configurações apropriadas

Teste o middleware da lista de permissões de IP:

Se sua solicitação vier de um IP que está na lista de permissões (por exemplo, 127.0.0.1), ela deverá ser bem-sucedida:
```bash
curl -k -I -H "Host: whoami.docker.localhost" https://localhost/
```
Se você tentar acessar de um IP que não esteja na lista de permissões, a solicitação será rejeitada com uma 403resposta "Proibido".
Para simular isso em um ambiente local, você pode modificar temporariamente a configuração do middleware para excluir seu endereço IP e testar novamente.