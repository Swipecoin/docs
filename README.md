Documentação Stellar
============

Neste repositório se encontra a documentação Stellar. Estes documentos alimentam o [builder do site Stellar para desenvolvedores](https://github.com/stellar/developers) e estão expostos em [stellar.org/developers](https://www.stellar.org/developers/).

## Como escrever documentos

Há algumas convenções ao escrever documentos que vão para o site de Stellar para Desenvolvedores. A maioria dos documentos são escritos em formato Markdown. Para uma introdução ao Markdown, dê uma olhada no [Guia do Github "Mastering Markdown"](https://guides.github.com/features/mastering-markdown/).

### Headers

Lines that start with a hash symbol (`#`) are considered headers. Below is an example of a few and the name of the header:

```
# h1
## h2
### h3
#### h4
```

- DO NOT use h1 since that is reserved for the page title generated from front matter.
- DO NOT skip a header size (don't go from h2 to h4).
- DO use smaller headers (more hash symbols) to represent that a section is nested under a parent one with a larger header.
- DO add a space after the hash symbols. Some Markdown parsers will not render the text as a header without the space.

### Front matter

At the top of most documents is something called "front matter". This is metadata for the file written in [YAML format](https://en.wikipedia.org/wiki/YAML).

Here is an example of front matter in action:
```
---
title: Horizon Reference
---
```

The currently used keys in the front matter are:
- title

### Document title

Do not start a page with a Markdown header (`# Title`). Instead, leave it in the front matter. The developers site will take the title from the front matter.

### Links

Use inline links and not reference links.

There are three different kinds of links, and each different kind of link has a significant meaning. Some of these links are transformed in the generation of the developers site.

<table>
  <tbody>
    <tr>
      <th>link type</th>
      <th>where to use</th>
      <th>markdown link example</th>
      <th>resulting link (after dev portal processing)</th>
    </tr>
    <tr>
    <tr>
      <td>Relative</td>
      <td><ul><li>links within same repository</li></ul></td>
      <td>../reference/accounts-all.md</td>
      <td>../reference/accounts-all.html</td>
    </tr>
    <tr>
      <td>Root relative</td>
      <td><ul><li>when you want to use the GitHub file viewer (e.g. for source files)</li></ul></td>
      <td>/src/ledger/AccountFrame.cpp</td>
      <td>https://github.com/stellar/CURRENT-REPOSITORY/tree/master/src</td>
    </tr>
    <tr>
      <td>Absolute links</td>
      <td>
        <ul>
          <li>cross repository links (should link to the dev portal at www.stellar.org/developers/)</li>
          <li>links to external sites (like https://www.google.com/)</li>
        </ul>
      </td>
      <td>https://www.stellar.org/developers/js-stellar-base/learn/building-transactions.html</td>
      <td>https://www.stellar.org/developers/js-stellar-base/learn/building-transactions.html</td>
    </tr>
  </tbody>
</table>

### Non-markdown files

Sometimes we want to include other types of content such as `.pdf`'s. To add front matter to the PDF, create a sibling file with the PDF file name and an added extension of `.metadata`. This file can then define metadata for the title of the `.pdf`.

An example can be seen in [stellar-core's software folder](https://github.com/stellar/stellar-core/tree/master/docs/software).

## Contributing

Your contributions to the Stellar network will help improve the world’s financial infrastructure, faster.

We want to make it as easy as possible to contribute changes that help the Stellar network grow and thrive. There are a few guidelines that we ask contributors to follow so that we can merge your changes quickly. Please read our [Contribution Guide](https://github.com/stellar/docs/blob/master/CONTRIBUTING.md) and sign our [Contributor License Agreement](https://docs.google.com/forms/d/1g7EF6PERciwn7zfmfke5Sir2n10yddGGSXyZsq98tVY/viewform).
