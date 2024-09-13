# Como Resolver o Cloudflare com Playwright em 2024 

![](https://assets.capsolver.com/prod/posts/cloudflare-playwright/yTIPss31ewhc-89c5f07b2908e89e2da788da09e3dcd8.png)

Sabe, existe uma certa emoção em superar obstáculos, especialmente quando esses obstáculos são guardiões digitais como o Cloudflare. Se você já se viu olhando para um desafio do Cloudflare enquanto tentava automatizar uma tarefa na web, você está em boa companhia. Já estive lá, muitas vezes. Mas em 2024, o jogo mudou e as ferramentas também. Deixe-me guiá-lo por como tenho lidado com o Cloudflare com o Playwright, e sim, também falaremos sobre o novo e furtivo na área, o Cloudflare Turnstile.

## O que é o Cloudflare e por que ele importa
Antes de mergulharmos no cerne da resolução de desafios do Cloudflare, vamos dedicar um momento para entender o que estamos enfrentando. O Cloudflare é um serviço de segurança robusto usado por milhões de sites para se proteger de tráfego malicioso, ataques DDoS e uma variedade de outras ameaças. Quando ele detecta um comportamento incomum - como um script automatizado tentando acessar uma página - ele lança um desafio, geralmente na forma de um CAPTCHA, para verificar se você é um humano e não um bot.

Mas aqui está o ponto principal: o Cloudflare não se limita a lançar CAPTCHAs simples. Em 2024, eles lançaram algo chamado Cloudflare Turnstile, um sistema de desafio mais sofisticado e adaptativo que é projetado para ser ainda mais resistente à automação. É uma tarefa difícil, mas com a abordagem certa, você ainda pode sair por cima.

![](https://assets.capsolver.com/prod/posts/cloudflare-playwright/JNvpoRi47OYh-a432a4e7015b3706f3600cf4efb1ec40.gif)

> Está lutando com a falha repetida em resolver completamente o irritante captcha?
>
> Descubra a resolução automática perfeita de captchas com a tecnologia **Capsolver** Auto Web Unblock, alimentada por IA!
>
> Reivindique seu   <u>**Código de Bônus**</u> para as melhores soluções de captcha; [CapSolver](https://www.capsolver.com/?utm_source=official&utm_medium=blog&utm_campaign=cloudflareplaywright): **WEBS**. Após resgatá-lo, você receberá um bônus extra de 5% após cada recarga, Ilimitado
> 
> ![](https://assets.capsolver.com/prod/images/post/2024-03-29/fbc29472-886c-45b2-9eb2-2b307f6d9700.png)


## Por que o Playwright é a ferramenta de escolha em 2024

Você pode estar se perguntando: "Por que o Playwright? Por que não ficar com o bom e velho Selenium ou Puppeteer?" E essa é uma pergunta justa. A resposta é que o Playwright emergiu como uma potência para automação da web, oferecendo recursos que o tornam particularmente eficaz contra desafios modernos como os apresentados pelo Cloudflare.

O Playwright oferece suporte a vários contextos de navegador, o que significa que você pode simular diferentes usuários de forma mais eficaz. Ele também fornece mais controle sobre o comportamento do navegador, facilitando a imitação de interações reais do usuário - algo crucial ao lidar com as medidas de segurança avançadas do Cloudflare.


## Começando: configurando o Playwright

Primeiro, se você ainda não o fez, precisará instalar o Playwright. Configurá-lo é direto:

```bash
npm install playwright
```

Depois de instalado, você está pronto para começar a automatizar suas tarefas na web. Mas se seu objetivo é superar os desafios do Cloudflare, especialmente seu novo CAPTCHA Turnstile, precisaremos dar alguns passos extras. Usaremos **[CapSolver](https://www.capsolver.com/?utm_source=official&utm_medium=blog&utm_campaign=cloudflareplaywright)**, uma API de terceiros projetada para resolver CAPTCHAs como o Turnstile, e integrá-la ao Playwright para acessar sites protegidos pelo Cloudflare.

### Passo 1: pegando o SiteKey

O primeiro obstáculo que você enfrentará com o CAPTCHA Turnstile é obter o `siteKey` da página da web. Essa chave é essencial para o CapSolver processar o CAPTCHA e fornecer um token válido.

Você pode extrair o `siteKey` inspecionando a fonte da página da web ou, para facilitar a vida, pode usar a **Extensão CapSolver**. Ela detecta automaticamente os parâmetros do CAPTCHA na página. Para um guia detalhado sobre como configurar isso, consulte nosso post de blog:  
[Identifique os parâmetros do Cloudflare Turnstile](https://www.capsolver.com/blog/Cloudflare/identify-cloudflare-turnstile-parameters/?utm_source=official&utm_medium=blog&utm_campaign=cloudflareplaywright).

Assim que você tiver o `siteKey`, estará pronto para avançar para a próxima etapa.

### Passo 2: Chamando a API CapSolver para resolver o CAPTCHA

Com o `siteKey` em mãos, é hora de usar a API do CapSolver para resolver o CAPTCHA Turnstile e recuperar um token válido. Esse token nos permitirá contornar o desafio e prosseguir com nossas tarefas de scraping ou automação na web.

Aqui está um trecho de código de amostra usando **axios** e **Playwright** para interagir com o CapSolver:

```javascript
const axios = require('axios');
const playwright = require("playwright");

const api_key = "YOUR_API_KEY"; // Sua chave de API CapSolver
const site_key = "0xxxxxx"; // O siteKey que você recuperou
const site_url = "https://xxx.xxx.xxx/xxx"; // O URL do site de destino
const proxy = "http://xxx:xxx@x.x.x.x:x"; // Opcional: Use seu proxy se necessário

async function solveCaptcha() {
  const payload = {
    clientKey: api_key,
    task: {
      type: 'AntiTurnstileTaskProxyLess',
      websiteKey: site_key,
      websiteURL: site_url,
      metadata: {
        action: '', // Opcional, especifique se necessário
        type: "turnstile"
      }
    }
  };

  try {
    const res = await axios.post("https://api.capsolver.com/createTask", payload);
    const task_id = res.data.taskId;
    if (!task_id) {
      console.log("Falha ao criar tarefa:", res.data);
      return;
    }

    console.log("Tarefa criada, aguardando token...");

    while (true) {
      await new Promise(resolve => setTimeout(resolve, 1000)); // Aguardar 1 segundo antes de verificar novamente
      const getResultPayload = {clientKey: api_key, taskId: task_id};
      const resp = await axios.post("https://api.capsolver.com/getTaskResult", getResultPayload);
      
      if (resp.data.status === "ready") {
        console.log("CAPTCHA resolvido, token recebido:", resp.data.solution.token);
        return resp.data.solution.token;
      }

      if (resp.data.status === "failed" || resp.data.errorId) {
        console.log("Falha ao resolver o CAPTCHA! Resposta:", resp.data);
        return;
      }
    }
  } catch (error) {
    console.error("Erro ao resolver o CAPTCHA:", error);
  }
}
```

Neste código, criamos uma tarefa enviando uma solicitação POST para a API do CapSolver, passando o `siteKey` e o URL do site que queremos acessar. Depois que a tarefa é criada, verificamos continuamente o status até que o CapSolver retorne um token de solução. Esse token é o que usaremos para provar ao Cloudflare que somos humanos.

### Passo 3: Injetando o token CAPTCHA com o Playwright

Agora que temos o token CAPTCHA, precisamos injetá-lo na sessão como um cookie usando o Playwright. Isso nos permitirá navegar pelo site sem ser bloqueados pela proteção do Cloudflare. Aqui está como fazer isso:

```javascript
const wait = (ms) => new Promise(resolve => setTimeout(resolve, ms));

async function accessSiteWithToken(){
  let clearanceCookie;

  // Resolver o CAPTCHA e obter o token
  await solveCaptcha().then(token => {
    clearanceCookie = token;
  });

  const browser = await playwright.chromium.launch();
  const context = await browser.newContext();
  const page = await context.newPage();

  await wait(500);

  // Injetar o token como um cookie
  await page.setCookie({
    name: "cf_clearance",
    value: clearanceCookie,
    url: site_url, // Certifique-se de que isso corresponda ao URL de destino
    domain: "xx.xx.xx" // Ajuste o domínio de acordo com o site real
  });

  await wait(500);

  // Navegar para o site após definir o cookie
  await page.goto(site_url);
  
  // Agora você pode fazer scraping do conteúdo ou interagir com a página livremente
  console.log("Acesso ao site realizado com sucesso!");

  await browser.close();
}

// Executar o script para acessar o site
accessSiteWithToken().then();
```

## Considerações finais

O Cloudflare, sem dúvida, tornou mais difícil fazer scraping de sites ou automatizar tarefas em 2024, mas com ferramentas como Playwright e CapSolver, o desafio está longe de ser impossível. A capacidade do Playwright de simular interações reais do usuário combinada com a API de resolução de CAPTCHA do CapSolver fornece uma maneira poderosa de contornar essas barreiras sem suar muito.

Claro, sempre é uma boa ideia garantir que você esteja dentro dos limites das práticas de scraping legais e éticas. Alguns sites têm políticas rigorosas em relação ao acesso automatizado, por isso certifique-se de que você esteja ciente delas antes de prosseguir.

No mundo em constante evolução da automação da web, tudo se resume a estar à frente da curva - e com o Playwright e o CapSolver, você está equipado para fazer exatamente isso.
