# Memed Sinapse Prescrição

Sinapse Prescrição é a ferramenta 100% gratuita da Memed que permite o seu site/plataforma/prontuário ter uma prescrição inteligente com diversas funcionalidades:

- Protocolos - Praticidade e velocidade na rotina de atendimentos.
- Minhas Posologias - Agilidade para o médico prescrever com base em seus histórico.
- Impressão inteligente - Regras de impressão, controle de margem, logo customizado e impressão automática de receituários especiais.
- Envio por SMS - Facilita a vida do paciente, entregando agilidade e permitindo a comparação de preços.
- Banco de medicamentos - O mais completo do Brasil, com informações constantemente curadas pela nossa equipe.

## Demonstração

Clique na imagem abaixo para ver um exemplo de integração:

<p align="center"><a href="https://memeddev.github.io/sinapse/demo-sinapse-prescricao.html" target="_blank"><img src="https://user-images.githubusercontent.com/2197005/38443811-53d52ab6-39c3-11e8-9f7a-a1fb8e6aeafd.png" alt="Memed Sinapse Prescrição Demo" /></a></p>

## Como integrar
A integração é feita em dois momentos: implementando o cadastro do usuário (profissional da saúde com CRM) via API e depois implementando o Sinapse Prescrição (front-end) dentro de seu sistema.
Logo no primeiro acesso do usuário, o mesmo deverá aceitar os termos de condições e então poderá acessar o Sinapse Prescrição.

### Chaves de acesso
A Memed disponibiliza duas chaves de acesso:

- **API-KEY** - Chave pública, pode aparecer no front-end. Permite a aplicação acessar os serviços de busca e cadastro de usuário.
- **SECRET-KEY** - Chave privada, deve ficar somente no back-end. Quando usada junto com a API-KEY, permite o envio de prescrições.

### Servidor para testes/homologação
Caso queira fazer um teste de integração, disponibilizamos um servidor com um banco de dados reduzido. Para utilizá-lo, substitua os domínios pelo dos servidores de teste:

| Domínio de produção | Domínio de teste |
|---------------------|------------------|
|memed.com.br|integracao.memed.com.br|
|api.memed.com.br|integracao.api.memed.com.br|

Para acessar alguns recursos na API de teste, utilize as chaves abaixo:

|   | Chaves de acesso |
|---|--------------------|
|API-KEY|iJGiB4kjDGOLeDFPWMG3no9VnN7Abpqe3w1jEFm6olkhkZD6oSfSmYCm|
|SECRET-KEY|Xe8M5GvBGCr4FStKfxXKisRo3SfYKI7KrTMkJpCAstzu2yXVN4av5nmL|

> **Atenção**: O banco de dados é limpo após alguns dias. Não envie qualquer dado sensível nesse servidor, outras pessoas irão acessá-lo.

### Integrando com a API
Para que o usuário (profissional da saúde com CRM) possa utilizar a prescrição da Memed, é necessário cadastrá-lo no banco de dados da Memed, seguindo o fluxo abaixo:

- O parceiro envia um request (especificado mais abaixo) com os dados do usuário para a Memed
- A Memed responde com um token e um ID de usuário
- O parceiro armazena o token e o ID em seu banco, para usar futuramente nas integrações

Exemplo do request usando cURL:
```curl
curl -X POST \
  'https://api.memed.com.br/v1/sinapse-prescricao/usuarios?api-key=API-KEY&secret-key=SECRET-KEY' \
  -H 'Accept: application/vnd.api+json' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{
     "data": {
       "type": "usuarios",
         "attributes": {
           "nome": "José",
           "data_nascimento": "11/11/2011",
           "sobrenome": "da Silva",
             "cpf": "000.000.000-00",
             "email": "meu@email.com.br",
             "uf": "SP",
             "sexo": "M",
             "crm": "6231232249"
           },
           "relationships": {
             "cidade": {
                "data": { "type": "cidades", "id": "5213" }
              },
             "especialidade": {
                "data": { "type": "especialidades", "id": "50" }
              }
           }
      }
 }'
```

Veja abaixo uma descrição de cada atributo, TODOS SÃO OBRIGATÓRIOS:
```
Atributos do usuário:

nome - Nome do usuário
sobrenome - Sobrenome do usuário
data_nascimento - Data de nascimento do usuário, no padrão brasileiro (DD/MM/YYYY)
cpf - CPF do usuário, com pontuação (XXX.XXX.XXX-XX)
crm - CRM do usuário, somente números
uf - UF do usuário (onde o CRM está cadastrado)
email - E-mail do usuário
sexo - Sexo do usuário (M ou F)

Relacionamentos do usuário:

cidade (relationships.cidade.id) - ID da cidade que o usuário atende
especialidade (relationships.especialidade.id) - ID da especialidade do usuário
```

Para encontrar os IDs da cidade e especialidade, é necessário fazer dois requests:

Lista de todas as especialidades
```
curl -X GET -H "Accept: application/json" "https://api.memed.com.br/v1/especialidades"
```

Lista de cidades, filtradas por um nome
```
curl -X GET -H "Accept: application/json" "http://api.memed.com.br/v1/cidades?filter[q]=Campinas"
```

#### Recuperando o token de um usuário

É possível ver todas as informações de um usuário, incluindo o token, através do request abaixo:

```curl
curl -X GET \
  'https://api.memed.com.br/v1/sinapse-prescricao/usuarios/CRM-DO-USUARIO?api-key=API-KEY&secret-key=SECRET-KEY' \
  -H 'Accept: application/vnd.api+json'
```

```
Parâmetros da URL:

CRM-DO-USUARIO - Número + UF (ex: "123456SP")
```

O retorno do request conterá os dados:

```json
{
    "data": {
        "type": "usuarios",
        "attributes": {
            "nome": "João",
            "sobrenome": "da Silva",
            "cpf": "00000000000",
            "email": "joao@dasilva.com.br",
            "data_nascimento": "01/01/1900",
            "sexo": "M",
            "cidade": "São Paulo",
            "crm": "123456",
            "especialidade": "Cirurgia oncológica",
            "token": "TOKEN_DO_USUARIO",
            "uf": "SP",
            "user_type": "Medicos",
            "total_of_prescriptions": 0,
            "total_of_prescripted_drugs": 0,
            "total_of_sms_prescriptions": 0
        },
        "links": {
            "self": "https://api.memed.com.br/v1/sinapse-prescricao/usuarios/123456SP"
        },
        "relationships": {
            "cidade": {
                "data": {
                    "id": 5213,
                    "type": "cidades"
                },
                "links": {
                    "self": "https://api.memed.com.br/v1/sinapse-prescricao/usuarios/123456SP/relationships/cidade"
                }
            },
            "sociedades": {
                "data": [],
                "links": {
                    "self": "https://api.memed.com.br/v1/sinapse-prescricao/usuarios/123456SP/relationships/sociedades"
                }
            },
            "especialidade": {
                "data": {
                    "id": 50,
                    "type": "especialidades"
                },
                "links": {
                    "self": "https://api.memed.com.br/v1/sinapse-prescricao/usuarios/123456SP/relationships/especialidade"
                }
            }
        },
        "id": 123456
    },
    "links": {
        "self": "https://api.memed.com.br/v1/sinapse-prescricao/usuarios/123456SP"
    }
}
```

## Integrando com o front-end
Para integrar o módulo de prescricão em sua plataforma é necessário definir um elemento com id `memed-prescricao` e depois incluir o script que irá carregar as dependências necessárias:

```
<a href="#" id="memed-prescricao">Prescrever</a>

<script
    type="text/javascript"
    src="http://memed.com.br/modulos/plataforma.sinapse-prescricao/build/sinapse-prescricao.min.js"
    data-token="TOKEN_DO_USUARIO_OBTIDO_NO_CADASTRO_VIA_API"
    data-color="#576cff">
</script>
```

Com a propriedade `data-color` você pode customizar a cor primária, deixando o Sinapse Prescrição mais parecido com o seu produto.

### Mostrando/Escondendo manualmente o módulo de prescrição

```js
// Mostra o módulo de prescrição
MdHub.module.show('plataforma.prescricao');
```

```js
// Esconde o módulo de prescrição
MdHub.module.hide('plataforma.prescricao');
```

### Definindo o paciente
Para definir o paciente, basta disparar um comando para o módulo de prescrição, como no exemplo abaixo:

```js
MdHub.command.send('plataforma.prescricao', 'setPaciente', {nome: 'José da Silva'});
```

Obs: A Memed identifica os pacientes pelo nome. Em caso de pacientes com nomes iguais, eles serão agrupados. Estamos desenvolvendo uma solução para coletar outros identificadores (cpf, rg...) de forma criptografada junto aos parceiros.

### Escutando eventos
Você pode assinar eventos que são disparados pelos módulos e implementa-los conforme necessidade em sua plataforma.

#### Quando o módulo de prescrição terminou de carregar
Os módulos da Memed emitem eventos quando inicializados, podendo ser capturados da seguinte forma:

```js
MdHub.event.add('core:moduleInit', function moduleInitHandler(module) { 
  if (module.name === 'plataforma.prescricao') {
    // Módulo de prescrição inicializado
  } 
});
```

#### Quando o módulo de prescricão for fechado
```js
MdSinapsePrescricao.event.add(
    'core:moduleHide',
    function moduloFechado(modulo) {
        if(modulo.moduleName === 'plataforma.prescricao') {
            console.log('====== Módulo fechado ======', modulo);
        }
    }
);
```

#### Quando um medicamento for adicionado
Caso queira capturar os dados do medicamento inserido, você pode adicionar um callback javascript:
```js
MdSinapsePrescricao.event.add('medicamentoAdicionado', function callback(medicamento) {
  // O objeto medicamento:
  // {
  //    "alto_custo":false,
  //    "composicao":"Princípio Ativo 1 + Princípio Ativo 2",
  //    "controle_especial":false,
  //    "descricao":"Ácido Ascórbico",
  //    "fabricante":"Sundown Vitaminas",
  //    "forma_fisica":"Cápsula",
  //    "id":"a123123123",
  //    "nome":"Vitamina C, comprimido (100un)",
  //    "quantidade":1,
  //    "tipo":"dermocosmético",
  //  }
});
```
## Capturando uma prescrição

Quando o usuário terminar uma prescrição, é possível capturar o ID da mesma, assim como pedir mais informações, através de um evento:

```javascript
MdHub.event.add('prescricaoSalva', function prescricaoSalvaCallback(idPrescricao) {
	// Aqui é possível enviar esse ID para seu back-end obter mais informações
	funcaoParaEnviarParaBackendObterMaisInformacoes(idPrescricao);
});
```

No back-end, para obter mais informações sobre a prescrição, basta enviar um request para a API da Memed, com o ID da prescrição e o token do usuário:

```bash
curl -X GET 'https://api.memed.com.br/v1/prescricoes/AQUI_VAI_O_ID_DA_PRESCRICAO?token=AQUI_VAI_O_TOKEN_DO_USUARIO' -H 'accept: application/json'
```

A resposta será como a abaixo:

```json
{
    "data": {
        "type": "prescricoes",
        "attributes": {
            "data": "11/04/2017",
            "horario": "17:34:32",
            "medicamentos": [
                {
                    "id": "d2916",
                    "nome": "Creme Corretivo Dia, creme (40mL)",
                    "descricao": "Melanostatine + Vitamina PP + Derivados de vitamina C",
                    "posologia": "<p>Usar 1x ao dia</p>",
                    "quantidade": 1,
                    "fabricante": "Topicrem",
                    "titularidade": "Dermocosmético",
                    "tipo": "dermocosmético",
                }
            ],
            "paciente": {
                "id": 64414,
                "nome": "Gabriel",
            },
        },
        "id": 1234
    }
}
```

## Capturando históricos de prescrições

```bash
curl -X GET \
  'https://api.memed.com.br/v1/prescricoes?token=AQUI_VAI_O_TOKEN_DO_USUARIO' \
  -H 'Accept: application/vnd.api+json'
```

O histórico de prescrições é páginado (10 em 10 itens). Caso deseje ver os próximos 10 itens:

```bash
curl -X GET \
  'https://api.memed.com.br/v1/prescricoes?token=AQUI_VAI_O_TOKEN_DO_USUARIO&page[offset]=10' \
  -H 'Accept: application/vnd.api+json'
```

## Trocando de usuário

Caso sua aplicação seja do tipo SPA (Single Page Application) e não faça "refresh" após a ação de login/logout, você pode redefinir o token do usuário logado utilizando o código abaixo:

```js
MdSinapsePrescricao.setToken('TOKEN_DO_NOVO_USUARIO');
```

Obs: A função acima causará o recarregamento dos iframes. A Memed somenta suporta os navegadores modernos (IE 11+), verifique se para os usuários do seu sistema, a atualização dos iframes ocorre corretamente.

## Cadastrando protocolos para o usuário

É possível cadastrar protocolos de tratamento para o usuário através da API:

```bash
curl -X POST \
  'https://api.memed.com.br/v1/protocolos?token=AQUI_VAI_O_TOKEN_DO_USUARIO' \
  -H 'Accept: application/vnd.api+json' \
  -H 'Content-Type: application/json' \
  -d '{
  "data": {
   "type": "protocolos",
   "attributes": {
     "nome": "Nome do Protocolo",
     "medicamentos": [
       {
         "nome": "Texto livre que não é um medicamento",
         "posologia": "<p>Tomar 2x ao dia</p>",
         "quantidade": "1",
         "composicao": null,
         "fabricante": null,
         "titularidade": null,
         "preco": null
       },
       {
         "id": "a61931095900",
         "nome": "ABC 10mg/g, Creme tópico (1un de 20g)",
         "posologia": "<p>Tomar 1x durante a noite por 15 dias</p>",
         "quantidade": "1",
         "composicao": "Clotrimazol 10mg/g",
         "fabricante": "Kley Hertz",
         "titularidade": "Similar",
         "preco": null
       }
     ]
   }
  }
}'
```

## Upload de template para um médico

É possível customizar o cabeçalho e rodapé da impressão da prescrição de um médico. Para isso, criamos um endpoint para que você possa fazer o upload dessas informações do para o seu médico. Basta chamar o endpoint passando um arquivo PDF como template. O arquivo PDF precisa possuir um cabeçalho e rodapé. Veja aqui um exemplo: [Template Exemplo](files/exemplo-template.pdf)

### Requisição de exemplo:
```bash
curl -X POST \
  'http://api.memed.com.br/v1/opcoes-receituario/upload-template?token=AQUI_VAI_O_TOKEN_DO_USUARIO' \
  -H 'Accept: application/vnd.api+json' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'content-type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW' \
  -F template=template-file.pdf
```

Perceba que nessa requisição, você vai precisar enviar um parâmetro `template` que contém um arquivo do tipo PDF. Em caso de sucesso, a API irá retornar um JSON contendo a imagem de header e footer que foram geradas e serão usadas na hora da impressão da prescrição médica.

### Exemplo de resposta:
```json
{
    "header": "https://memed.s3-sa-east-1.amazonaws.com/parceiros/templates/imagem-header.jpeg",
    "footer": "https://memed.s3-sa-east-1.amazonaws.com/parceiros/templates/imagem-footer.jpeg"
}
```
