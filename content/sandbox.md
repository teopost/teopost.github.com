+++
banner = "sandbox/sample.jpg"
sample = "fake.pdf"
+++

## Note

This is a footnote.[^1]

[^1]: the footnote text.

## Tabelle

Name    | Age
--------|------
Bob     | 27
Alice   | 23

## Task list

- [ ] a task list item
- [ ] list syntax required
- [ ] incomplete
- [x] completed

## Esempi di link:

* Link a una pagina esterna: [Sergio](https://www.sergiogridelli.it/)
* Link a un articolo del blog: [LAMP server]({{< ref "installare-lamp-raspberry.md" >}})

## Esempio di quote

> Nota quotata

## Esempi di shortcodes
https://gohugo.io/content-management/shortcodes/


{{< highlight html >}}
<section id="main">
  <div>
   <h1 id="title">{{ .Title }}</h1>
    {{ range .Pages }}
        {{ .Render "summary"}}
    {{ end }}
  </div>
</section>
{{< /highlight >}}

---

### ref
[GoDaddy]({{< ref "post/2016-12/configurare-godaddy-per-le-github-pages.md" >}})


### notice

{{< highlight sql >}} A bunch of code here {{< /highlight >}}

{{% notice note %}}
A notice disclaimer
{{% /notice %}}

{{% notice info %}}
A notice disclaimer
{{% /notice %}}

{{% notice tip %}}
A notice disclaimer
{{% /notice %}}

{{% notice warning %}}
A notice disclaimer
{{% /notice %}}

{{< notice warning >}}
This is a warning notice. Be warned!
{{< /notice >}}

### categories

- fun
- work

### mermaid

{{<mermaid>}}
graph LR;
  A-->B;
  A-->C;
  B-->D;
  C-->D;
{{</mermaid>}}

due

{{<mermaid>}}
graph LR;
  A-->B;
  A-->C;
{{</mermaid>}}



### gallery

{{< gallery
    "/apache-pagina-cortesia/apache-pagina-cortesia.png"
    "/disabilitare-autenticazione-ubuntu-software-center/disabilitare-autenticazione-ubuntu-software-center.png"
    "/hotspot-wifi-con-raspberry-pi/hotspot-wifi-con-raspberry-pi.jpg"
>}}

### Buttons

{{% button href="https://github.com/teopost/pi-kiosk/" icon="fa fa-github" %}}Scarica Pi-Kiosk da Git-Hub{{% /button %}}

{{% button href="https://getgrav.org/" icon="fa fa-play" %}}Get Grav with icon{{% /button %}}
{{% button href="https://getgrav.org/" icon="fa fa-share" icon-position="right" %}}Get Grav with icon right{{% /button %}}
