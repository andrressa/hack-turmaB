# Fraude Lab — servidor para liberar desafios durante a aula

O projeto inclui o back-end, a página dos alunos, o painel da professora e os mesmos arquivos de dados do hackathon. Não precisa conectar ao site anterior: este servidor oferece a experiência completa em um único endereço.

## Como funciona na sala

1. Você abre https://SEU-SERVIDOR/professora e informa usuário e senha no diálogo do navegador.
2. Compartilha https://SEU-SERVIDOR/ com os alunos. Eles não precisam de senha.
3. A missão inicial aparece automaticamente, com histórico, pedidos novos e dicionário.
4. Você aperta “Liberar desafio 1”, depois 2, depois 3, no momento desejado.
5. As páginas abertas dos alunos recebem o comunicado e os downloads. A consulta ocorre a cada 2 segundos; abas em segundo plano ou conexão lenta podem demorar mais. Quem entra depois vê as rodadas já liberadas.
6. O campo de comunicados envia avisos para todos: pedir entregas, informar tempo ou esclarecer regras.
7. “Encerrar” mostra o aviso de fim; “Reabrir” retoma; “Reiniciar” prepara uma nova turma e bloqueia novamente as surpresas.

Os alunos devem manter a página deste servidor aberta. O HTML offline e o site privado anterior não recebem os comandos. Nenhum notebook é executado ou enviado neste servidor: a modelagem ocorre no Colab/Python, e você recolhe as entregas no canal da turma. Gabaritos e soluções continuam exclusivamente no pacote da professora já entregue; não estão neste servidor.

## Publicar no Render — passo a passo

Extraia o ZIP e coloque o CONTEÚDO da pasta fraude_backend em um repositório GitHub (package.json e server.js na raiz). Inclua public/, data/, config.js e render.yaml. Não envie .env nem runtime/.

No Render: New → Web Service → conectar esse repositório.

Configuração:
- Runtime: Node.
- Build Command: node --check server.js
- Start Command: node server.js
- Health Check Path: /health
- Environment Variables: NODE_VERSION = 24; ADMIN_USER = professora; ADMIN_PASSWORD = sua senha pessoal com pelo menos 12 caracteres; STATE_DIR = ./runtime.
- O Render define PORT automaticamente. Não coloque localhost no link da turma: use o endereço HTTPS público fornecido pelo Render.

Também é possível usar New → Blueprint com render.yaml. A senha será solicitada na configuração; não há senha pronta no código.

O modelo padrão usa o plano gratuito. O disco desse plano é temporário: o arquivo de liberações é perdido quando o serviço reinicia, fica inativo ou sofre redeploy. Antes da aula, abra o painel e confira o estado. Se houver reinicialização, os dados originais seguem no projeto, mas pode ser necessário liberar novamente as rodadas. Serviços gratuitos podem ficar inativos e demorar a responder na primeira abertura.

Para preservar o estado no Render, use um serviço com disco persistente e STATE_DIR=/var/data/fraude-lab. O arquivo render-persistente.yaml é uma alternativa paga, não ativada automaticamente. Para Blueprint, use esse arquivo como especificação ou substitua render.yaml por ele antes de publicar. Disco de 1 GB montado em /var/data. Confira o preço na sua conta antes de selecionar esse plano.

## Executar no seu computador (Windows)

Instale Node.js 24. Extraia o ZIP. Abra o terminal dentro da pasta fraude_backend.

Copie .env.example para .env, abra .env e escreva sua senha no ADMIN_PASSWORD, entre aspas. Mantenha a senha somente com você.

No PowerShell:

```powershell
Copy-Item .env.example .env
notepad .env
npm start
```

Abra http://localhost:3000/professora. O usuário padrão é professora. A página dos alunos é http://localhost:3000/.

Localhost só funciona no seu computador. Para uso em sala, prefira um servidor HTTPS; em rede local, seria necessário usar o IP do seu computador e permitir acesso à porta. O login da professora deve usar HTTPS em servidores públicos.

## Outro servidor / Docker

Precisa de Node.js 22.9 ou superior; foi testado com Node.js 24.19. Não há dependências externas. Execute node server.js com ADMIN_PASSWORD definido e PORT conforme o servidor. Use HTTPS na frente do aplicativo. STATE_DIR deve apontar para uma pasta gravável e persistente.

Docker:

```bash
docker build -t fraude-lab .
docker run --env-file .env -e STATE_DIR=/app/runtime -p 3000:3000 -v fraude_estado:/app/runtime fraude-lab
```

A pasta runtime contém estado.json. O volume preserva as liberações. O projeto usa uma única instância Node: não configure múltiplas réplicas compartilhando esse JSON. Dentro do mesmo processo, todos os alunos consultam o mesmo estado.

## Arquivos e regras

- Missão inicial: 60.000 linhas históricas, 5.000 pedidos novos e dicionário.
- Desafio 1: 8.000 compras da nova plataforma; até 40 investigações/dia.
- Desafio 2: 2.000 compras auditadas com rótulos + 8.000 compras futuras; até 40/dia.
- Desafio 3: 6.000 compras; até 15/dia; valor das fraudes selecionadas − R$ 50 por legítima selecionada.
- Total de registros: 89.000, incluindo 900 duplicatas didáticas no histórico.

Os CSVs e ZIPs futuros são bloqueados pelo back-end até a liberação. Não basta saber o endereço de download. A página pública não revela os enunciados futuros. Depois da liberação, os arquivos são acessíveis a quem tiver o link do servidor: este sistema não cadastra ou autentica equipes.

Reiniciar limpa os avisos e bloqueia as rodadas no servidor, mas não remove dados que os alunos já tenham baixado. Antes de uma nova turma, escolha “Reiniciar”. Depois de reiniciar o processo do servidor, atualize o painel da professora para renovar a autorização dos botões.

## Conferência

npm test

O teste verifica estado compartilhado entre consultas, liberação sequencial, proteção de downloads futuros, login, autorização de comandos, avisos, encerramento, reset e recuperação do estado após reinício. Os testes foram executados; a interface não foi inspecionada visualmente em navegador.

## Documentação consultada

https://render.com/docs/deploy-node-express-app
https://render.com/docs/disks
https://render.com/docs/free
https://render.com/docs/blueprint-spec
https://nodejs.org/api/http.html
