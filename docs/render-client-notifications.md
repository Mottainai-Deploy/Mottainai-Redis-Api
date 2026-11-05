# Redis de avisos do Cliente no Render

O Blueprint prepara dois serviços gratuitos na mesma região: um Render Key Value e a API HTTP dos avisos do Cliente. O endereço interno do Key Value é fornecido automaticamente à API em `REDIS_URL`.

## Antes de criar

- Confira se o workspace já possui uma instância gratuita de Key Value. O plano gratuito permite somente uma por workspace; caso exista uma instância destinada ao projeto, adapte a referência antes de sincronizar o Blueprint para evitar criação duplicada.
- O Blueprint usa Oregon, alinhado ao padrão do Blueprint inicial da API Cliente. Os serviços que acessam o endereço interno precisam estar no mesmo workspace e na mesma região.
- O Key Value aceita somente conexões internas (`ipAllowList: []`) e usa `noeviction`: ao atingir o limite de memória, as escritas falham em vez de remover itens silenciosamente.
- A instância gratuita não oferece persistência: reinicializações podem apagar filas e avisos. Monitore uso e disponibilidade durante a feira; esse formato não garante retenção permanente.

## Publicação

1. No Render, crie um Blueprint selecionando a branch que contém este `render.yaml` do repositório de implantação. Revise os dois recursos e confirme o plano gratuito de cada um.
2. O Render pedirá `JWT_SECRET` e `NOTIFICATION_PUBLISHER_SECRET`. Use os valores privados existentes, sem colocá-los no Git nem no chat. O primeiro deve ser Base64 e decodificar pelo menos 32 bytes; ele precisa ser igual a `CLIENT_NOTIFICATIONS_JWT_SECRET` da API Cliente. O segundo precisa ter pelo menos 32 caracteres e ser igual ao segredo de publicação da API Cliente.
3. Aguarde o Key Value e a API. Confira `GET /livez` e `GET /readyz`; o segundo verifica a conexão com o Key Value. Uma lista de avisos sem token deve ser rejeitada.
4. Copie a **URL HTTPS da API HTTP**, sem caminho adicional, para `CUSTOMER_NOTIFICATION_REDIS_URL` da API Cliente. Não use nesse campo a URL `redis://` do Key Value. Preencha as demais variáveis de notificações da API Cliente com os valores correspondentes.
5. Habilite emissão de tokens e disparo na API Cliente somente depois de revisar o Blueprint dela, configurar as chaves e garantir que a instância local não esteja disparando para o mesmo banco. Teste publicação, recebimento, leitura, limites e prioridade com contas autorizadas.

`autoDeployTrigger: off` mantém deploy automático por commit desativado; execute atualizações manualmente pelo painel depois da revisão. `/readyz` testa conexão, não comprova entrega de ponta a ponta.

Referências: https://render.com/docs/key-value e https://render.com/docs/free.
