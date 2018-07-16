# Como contribuir para um projeto Stellar

Suas contribuições à [rede Stellar](https://www.stellar.org/) vão ajudar a melhorar a infraestrutura financeira mundial mais rapidamente.

Queremos que seja o mais fácil possível para contribuir alterações que ajudem a rede Stellar a crescer e prosperar. Há algumas orientações que pedimos que os contribuidores sigam para podermos realizar merge com suas mudanças rapidamente.

## Para Começar

* Certifique-se de que você tem uma [conta no GitHub](https://github.com/signup/free).
* Crie uma issue no GitHub para sua contribuição, se já não houver.
  * Descreva claramente o problema, incluindo passos para reproduzí-lo casjo seja um bug.
* Faça um fork do repisitório no GitHub.

### Pequenas Mudanças

#### Documentação

Para pequenas alterações nos comentários e na documentação, nem sempre é preciso criar uma nova issue no GitHub. Nesse caso, é adequado começar a primeira linha de um commit com 'doc' em vez do número de uma issue.

## Encontrar coisas para trabalhar

O melhor lugar para começar é sempre dando uma olhada nas issues atuais no GitHub referentes ao projeto em que você tem interesse em contribuir. Issues marcadas com [help wanted](https://github.com/issues?q=is%3Aopen+is%3Aissue+user%3Astellar+label%3A%22help+wanted%22) costumam não ser muito abrangentes e são um bom lugar para começar.

O Stellar.org também usa essas mesmas issues do GitHub para acompanhar aquilo em que estamos trabalhando. Caso veja alguma issue atribuída a uma pessoa específica ou que tenha um rótulo `in progress`, significa que alguém está trabalhando nessa issue. O rótulo `orbit` quer dizer que provavelmente vamos trabalhar nessa issue nas próximas semanas. O rótulo `ready` significa que é uma issue prioritária e que trabalharemos nela na nossa próxima "orbit" (o termo que usamos no lugar de um sprint) ou na seguinte.

 E, é claro, sinta-se livre para criar uma nova issue se achar que algo precisa ser adicionado ou consertado.

## Making Changes

* Create a topic branch from where you want to base your work.
  * This is usually the master branch.
  * Please avoid working directly on the `master` branch.
* Make sure you have added the necessary tests for your changes and make sure all tests pass.

## Submitting Changes

* [Sign the Contributor License Agreement](https://docs.google.com/forms/d/1g7EF6PERciwn7zfmfke5Sir2n10yddGGSXyZsq98tVY/viewform?usp=send_form).
* All content, comments, and pull requests must follow the [Stellar Community Guidelines](https://www.stellar.org/community-guidelines/).
* Push your changes to a topic branch in your fork of the repository.
* Submit a pull request to the [docs repository](https://github.com/stellar/docs) in the Stellar organization.
 * Include a descriptive [commit message](https://github.com/erlang/otp/wiki/Writing-good-commit-messages).
 * Changes contributed via pull request should focus on a single issue at a time.
 * Rebase your local changes against the master branch. Resolve any conflicts that arise.

At this point you're waiting on us. We like to at least comment on pull requests within three
business days (typically, one business day). We may suggest some changes, improvements or alternatives.

# Additional Resources

* [Contributor License Agreement](https://docs.google.com/forms/d/1g7EF6PERciwn7zfmfke5Sir2n10yddGGSXyZsq98tVY/viewform?usp=send_form)
* [Explore the API](https://www.stellar.org/developers/reference/)
* #dev channel on [Slack](http://slack.stellar.org)
* #stellar-dev IRC channel on freenode.org

This document is inspired by:

* https://github.com/puppetlabs/puppet/blob/master/CONTRIBUTING.md
* https://github.com/thoughtbot/factory_girl_rails/blob/master/CONTRIBUTING.md
* https://github.com/rust-lang/rust/blob/master/CONTRIBUTING.md
