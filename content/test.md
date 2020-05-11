+++
banner = "banner/sample.jpg"
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

### notice

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

### gallery

{{< gallery
    "/banner/apache-pagina-cortesia.png"
    "/banner/disabilitare-autenticazione-ubuntu-software-center.png"
    "/banner/hotspot-wifi-con-raspberry-pi.jpg"
>}}

### Buttons

{{% button href="https://github.com/teopost/pi-kiosk/" icon="fa fa-github" %}}Scarica Pi-Kiosk da Git-Hub{{% /button %}}

{{% button href="https://getgrav.org/" icon="fa fa-play" %}}Get Grav with icon{{% /button %}}
{{% button href="https://getgrav.org/" icon="fa fa-share" icon-position="right" %}}Get Grav with icon right{{% /button %}}
