---
title: Solução de problemas de detecção de dependências vulneráveis
intro: 'Se as informações sobre dependências relatadas por {% data variables.product.product_name %} não são o que você esperava, há uma série de pontos a considerar, e várias coisas que você pode verificar.'
shortTitle: Detecção de solução de problemas
redirect_from:
  - /github/managing-security-vulnerabilities/troubleshooting-the-detection-of-vulnerable-dependencies
  - /code-security/supply-chain-security/troubleshooting-the-detection-of-vulnerable-dependencies
versions:
  fpt: '*'
  ghes: '*'
  ghae: issue-4864
  ghec: '*'
type: how_to
topics:
  - Dependabot
  - Alerts
  - Troubleshooting
  - Errors
  - Security updates
  - Dependencies
  - Vulnerabilities
  - Dependency graph
  - Alerts
  - CVEs
  - Repositories
---

{% data reusables.dependabot.beta-security-and-version-updates %}

Os resultados da detecção de dependências relatados pelo {% data variables.product.product_name %} podem ser diferentes dos resultados retornados por outras ferramentas. Existem boas razões para isso e é útil entender como {% data variables.product.prodname_dotcom %} determina as dependências para o seu projeto.

## Por que algumas dependências parecem estar faltando?

O {% data variables.product.prodname_dotcom %} gera e exibe dados de dependência de maneira diferente de outras ferramentas. Consequentemente, se você usou outra ferramenta para identificar dependências, quase certamente verá resultados diferentes. Considere o seguinte:

*   {% data variables.product.prodname_advisory_database %} é uma das fontes de dados que {% data variables.product.prodname_dotcom %} usa para identificar dependências vulneráveis. É um banco de dados gratuito e curado com informações sobre vulnerabilidade para ecossistemas de pacote comum em {% data variables.product.prodname_dotcom %}. Inclui tanto dados relatados diretamente para {% data variables.product.prodname_dotcom %} de {% data variables.product.prodname_security_advisories %} quanto os feeds oficiais e as fontes comunitárias. Estes dados são revisados e curados por {% data variables.product.prodname_dotcom %} para garantir que informações falsas ou não acionáveis não sejam compartilhadas com a comunidade de desenvolvimento. {% data reusables.security-advisory.link-browsing-advisory-db %}
*   O gráfico de dependências analisa todos os arquivos conhecidos de manifesto de pacote no repositório de um usuário. Por exemplo, para o npm, ele irá analisar o arquivo _package-lock.json_. Ele constrói um gráfico de todas as dependências do repositório e dependências públicas. Isso acontece quando você habilita o gráfico de dependências e quando alguém faz push para o branch-padrão, e inclui commits que fazem alterações em um formato de manifesto compatível. Para obter mais informações, consulte "[Sobre o gráfico de dependência](/github/visualizing-repository-data-with-graphs/about-the-dependency-graph)".
*   {% data variables.product.prodname_dependabot %} verifica qualquer push, para o branch-padrão, que contém um arquivo de manifesto. Quando um novo registro de vulnerabilidade é adicionado, ele verifica todos os repositórios existentes e gera um alerta para cada repositório vulnerável. {% data variables.product.prodname_dependabot_alerts %} são agregados ao nível do repositório, em vez de criar um alerta por vulnerabilidade. Para obter mais informações, consulte "[Sobre alertas para dependências vulneráveis](/code-security/supply-chain-security/about-alerts-for-vulnerable-dependencies)"
*   {% ifversion fpt or ghec or ghes > 3.2 %}{% data variables.product.prodname_dependabot_security_updates %} são acionados quando você recebe um alerta sobre uma dependência vulnerável no repositório. Sempre que possível, {% data variables.product.prodname_dependabot %} cria um pull request no repositório para atualizar a dependência vulnerável à versão mínima segura necessária para evitar a vulnerabilidade. Para obter mais informações, consulte "[Sobre {% data variables.product.prodname_dependabot_security_updates %}](/github/managing-security-vulnerabilities/about-dependabot-security-updates)" e "[Solução de problemas de {% data variables.product.prodname_dependabot %}](/github/managing-security-vulnerabilities/troubleshooting-dependabot-errors)".

    {% endif %}{% data variables.product.prodname_dependabot %} não pesquisa repositórios com relação a dependências vulneráveis de uma programação, mas o faz quando algo muda. Por exemplo, aciona-se uma varredura quando uma nova dependência é adicionada ({% data variables.product.prodname_dotcom %} verifica isso em cada push), ou quando uma nova vulnerabilidade é adicionada ao banco de dados da consultoria{% ifversion ghes or ghae-issue-4864 %} e sincronizado com {% data variables.product.product_location %}{% endif %}. Para obter mais informações, consulte "[Sobre alertas para dependências vulneráveis](/code-security/supply-chain-security/about-alerts-for-vulnerable-dependencies#detection-of-vulnerable-dependencies)"

## Por que não recebo alertas de vulnerabilidade em alguns ecossistemas?

O {% data variables.product.prodname_dotcom %} limita seu suporte a alertas de vulnerabilidade a um conjunto de ecossistemas onde podemos fornecer dados de alta qualidade e relevantes. As vulnerabilidades curadas no {% data variables.product.prodname_advisory_database %}, o gráfico de dependências, {% ifversion fpt or ghec %}{% data variables.product.prodname_dependabot %} atualizações de segurança, {% endif %}e  {% data variables.product.prodname_dependabot_alerts %} são fornecidos para vários ecossistemas, incluindo o Maven do Javaen, o npm do JavaScript e o Yarn, Nuget do .NET's, Pip Python, RubyGems e PHP Composer. Nós continuaremos a adicionar suporte para mais ecossistemas ao longo do tempo. Para uma visão geral dos ecossistemas de pacotes suportados por nós, consulte "[Sobre o gráfico de dependências](/github/visualizing-repository-data-with-graphs/about-the-dependency-graph#supported-package-ecosystems)".

Vale a pena observar que as consultorias de segurança de {% data variables.product.prodname_dotcom %} podem existir para outros ecossistemas. As informações em uma consultoria de segurança são fornecidas pelos mantenedores de um determinado repositório. Estes dados não são curados da mesma forma que as informações relativas aos ecossistemas suportados. {% ifversion fpt or ghec %}Para obter mais informações, consulte "[Sobre consultorias de segurança de {% data variables.product.prodname_dotcom %}](/github/managing-security-vulnerabilities/about-github-security-advisories){% endif %}

**Verificar**: A vulnerabilidade não capturada se aplica a um ecossistema não suportado?

## O gráfico de dependências só encontra dependências nos manifestos e nos arquivos de bloquei?

O gráfico de dependências inclui informações sobre dependências explicitamente declaradas em seu ambiente. Ou seja, dependências que são especificadas em um manifesto ou um arquivo de bloqueio. O gráfico de dependências, geralmente, também inclui dependências transitivas, mesmo quando não são especificadas em um arquivo de travamento analisando as dependências das dependências em um arquivo de manifesto.

Os {% data variables.product.prodname_dependabot_alerts %} aconselham você com relação a dependências que você deve atualizar, incluindo dependências transitivas, em que a versão pode ser determinada a partir de um manifesto ou de um arquivo de bloqueio. {% ifversion fpt or ghec or ghes > 3.2 %}{% data variables.product.prodname_dependabot_security_updates %} sugere apenas uma mudança em que {% data variables.product.prodname_dependabot %} pode "corrigir" diretamente a dependência, ou seja, quando são:
* Dependências diretas, que são definidas explicitamente em um manifesto ou arquivo de bloqueio
* Dependências transitórias declaradas em um arquivo de bloqueio{% endif %}

O gráfico de dependências não inclui dependências de "soltas". Dependências "soltas" são arquivos individuais copiados de outra fonte e verificados no repositório diretamente ou dentro de um arquivo (como um arquivo ZIP ou JAR), em vez de ser referenciadas pelo manifesto ou arquivo de bloqueio do gerenciador de pacotes.

**Verifique**: A vulnerabilidade não detectada para um componente não especificado no manifesto ou no arquivo de bloqueio do repositório?

## O gráfico de dependências detecta dependências especificadas usando variáveis?

O gráfico de dependências analisa como são carregados para {% data variables.product.prodname_dotcom %}. O gráfico de dependência não tem acesso ao ambiente de construção do projeto. Portanto, ele não pode resolver variáveis usadas dentro dos manifestos. Se você usar variáveis dentro de um manifesto para especificar o nome, ou mais comumente, a versão de uma dependência, essa dependência não será incluída no gráfico de dependências.

**Verifique**: A dependência ausente é declarada no manifesto usando uma variável para seu nome ou versão?

## Existem limites que afetam os dados do gráfico de dependências?

Sim, o gráfico de dependências tem duas categorias de limites:

1. **Limites de processamento**

    Eles afetam o gráfico de dependências exibido dentro de {% data variables.product.prodname_dotcom %} e também impedem que sejam criados {% data variables.product.prodname_dependabot_alerts %}.

    Manifestos com tamanho superior a 0.5 MB são processados apenas para contas corporativas. Para outras contas, manifestos acima de 0,5 MB são ignorados e não criarão {% data variables.product.prodname_dependabot_alerts %}.

    Por padrão, o {% data variables.product.prodname_dotcom %} não processará mais de 20 manifestos por repositório. {% data variables.product.prodname_dependabot_alerts %} não foi criado para manifestos acima deste limite. Se você precisar aumentar o limite, entre em contato com {% data variables.contact.contact_support %}.

2. **Limites de visualização**

    Eles afetam o que é exibido no gráfico de dependências dentro de {% data variables.product.prodname_dotcom %}. No entanto, eles não afetam {% data variables.product.prodname_dependabot_alerts %} que foram criados.

    A exibição de dependências do gráfico de dependências em um repositório só exibe 100 manifestos. De modo geral, isso é adequado, já que é significativamente maior do que o limite de processamento descrito acima. Em situações em que o limite de processamento é superior a 100, os {% data variables.product.prodname_dependabot_alerts %} são criados para quaisquer manifestos que não são mostrados dentro de {% data variables.product.prodname_dotcom %}.

**Verifique**: A dependência que falta está em um arquivo de manifesto superior a 0,5 MB ou em um repositório com um grande número de manifestos?

## O {% data variables.product.prodname_dependabot %} gera alertas de vulnerabilidades que são conhecidas há muitos anos?

O {% data variables.product.prodname_advisory_database %} foi lançado em novembro de 2019 e preencheu, inicialmente, a inclusão de informações de vulnerabilidade para os ecossistemas compatíveis a partir de 2017. Ao adicionar CVEs ao banco de dados, priorizamos a curadoria de CVEs mais recentes e CVEs que afetam versões mais recentes do software.

Algumas informações sobre vulnerabilidades mais antigas estão disponíveis, especialmente quando estes CVEs estão particularmente disseminados. No entanto algumas vulnerabilidades antigas não estão incluídas no {% data variables.product.prodname_advisory_database %}. Se houver uma vulnerabilidade antiga específica que você precisar incluir no banco de dados, entre em contato com {% data variables.contact.contact_support %}.

**Verifique**: A vulnerabilidade não detectada tem uma data de publicação anterior a 2017 no Banco de Dados Nacional de Vulnerabilidade?

## Por que o {% data variables.product.prodname_advisory_database %} usa um subconjunto de dados de vulnerabilidade publicada?

Algumas ferramentas de terceiros usam dados de CVE não descurados que não são verificados ou filtrados por um ser humano. Isto significa que os CVEs com erros de etiqueta ou de gravidade, ou outros problemas de qualidade, gerarão alertas mais frequentes, mais ruidosos e menos úteis.

Uma vez que {% data variables.product.prodname_dependabot %} usa dados curados em {% data variables.product.prodname_advisory_database %}, o volume de alertas pode ser menor, mas os alertas que você recebe serão precisos e relevantes.

{% ifversion fpt or ghec %}
## Cada vulnerabilidade de dependência gera um alerta separado?

Quando uma dependência tem várias vulnerabilidades, gera-se um alerta para cada vulnerabilidade no nível da consultoria mais manifesto.

![Captura de tela da aba de {% data variables.product.prodname_dependabot_alerts %} que mostra dois alertas do mesmo pacote com manifestos diferentes.](/assets/images/help/repository/dependabot-alerts-view.png)

O legado de {% data variables.product.prodname_dependabot_alerts %} foi agrupado em um único alerta agregado com todas as vulnerabilidades para a mesma dependência. Se você acessar um link para um alerta de legadode {% data variables.product.prodname_dependabot %}, você será redirecionado para a aba de {% data variables.product.prodname_dependabot_alerts %} filtrada para exibir vulnerabilidades para esse pacote e manifesto dependente.

![Screenshot of the {% data variables.product.prodname_dependabot_alerts %} tab showing the filtered alerts from navigating to a legacy {% data variables.product.prodname_dependabot %} alert.](/assets/images/help/repository/legacy-dependabot-alerts-view.png)

The {% data variables.product.prodname_dependabot_alerts %} count in {% data variables.product.prodname_dotcom %} shows a total for the number of alerts, which is the number of vulnerabilities, not the number of dependencies.

**Check**: If there is a discrepancy in the totals you are seeing, check that you are not comparing alert numbers with dependency numbers. Also check that you are viewing all alerts and not a subset of filtered alerts.
{% endif %}

## Leia mais

- "[Sobre alertas para dependências vulneráveis](/code-security/supply-chain-security/about-alerts-for-vulnerable-dependencies)"
- "[Visualizar e atualizar dependências vulneráveis no seu repositório](/github/managing-security-vulnerabilities/viewing-and-updating-vulnerable-dependencies-in-your-repository)"
- "[Gerenciar as configurações de segurança e análise do repositório](/github/administering-a-repository/managing-security-and-analysis-settings-for-your-repository)"{% ifversion fpt or ghec or ghes > 3.2 %}
- "[Solucionar problemas de {% data variables.product.prodname_dependabot %}](/github/managing-security-vulnerabilities/troubleshooting-dependabot-errors)"{% endif %}
