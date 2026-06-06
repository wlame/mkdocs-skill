# Color Schemes — Material for MkDocs

Full palette configurations for each supported organization color scheme.
Apply to the `theme.palette` section of `mkdocs.yml`.

---

## indigo (default)

Professional, trustworthy. Works well for internal tools, developer platforms, infrastructure.

```yaml
theme:
  palette:
    - media: "(prefers-color-scheme: light)"
      scheme: default
      primary: indigo
      accent: blue
      toggle:
        icon: material/brightness-7
        name: Switch to dark mode
    - media: "(prefers-color-scheme: dark)"
      scheme: slate
      primary: indigo
      accent: blue
      toggle:
        icon: material/brightness-4
        name: Switch to light mode
```

---

## teal

Modern, clean, technical. Good for APIs, CLIs, DevTools.

```yaml
theme:
  palette:
    - media: "(prefers-color-scheme: light)"
      scheme: default
      primary: teal
      accent: cyan
      toggle:
        icon: material/brightness-7
        name: Switch to dark mode
    - media: "(prefers-color-scheme: dark)"
      scheme: slate
      primary: teal
      accent: cyan
      toggle:
        icon: material/brightness-4
        name: Switch to light mode
```

---

## blue-grey

Muted, sophisticated. Good for enterprise software, serious tooling.

```yaml
theme:
  palette:
    - media: "(prefers-color-scheme: light)"
      scheme: default
      primary: blue-grey
      accent: blue
      toggle:
        icon: material/brightness-7
        name: Switch to dark mode
    - media: "(prefers-color-scheme: dark)"
      scheme: slate
      primary: blue-grey
      accent: light-blue
      toggle:
        icon: material/brightness-4
        name: Switch to light mode
```

---

## deep-purple

Creative, bold. Good for ML/AI tools, data science, creative platforms.

```yaml
theme:
  palette:
    - media: "(prefers-color-scheme: light)"
      scheme: default
      primary: deep purple
      accent: purple
      toggle:
        icon: material/brightness-7
        name: Switch to dark mode
    - media: "(prefers-color-scheme: dark)"
      scheme: slate
      primary: deep purple
      accent: purple
      toggle:
        icon: material/brightness-4
        name: Switch to light mode
```

---

## orange

Energetic, open-source feel. Good for community tools, open-source libraries.

```yaml
theme:
  palette:
    - media: "(prefers-color-scheme: light)"
      scheme: default
      primary: orange
      accent: amber
      toggle:
        icon: material/brightness-7
        name: Switch to dark mode
    - media: "(prefers-color-scheme: dark)"
      scheme: slate
      primary: orange
      accent: amber
      toggle:
        icon: material/brightness-4
        name: Switch to light mode
```

---

## Notes

- Always provide both light and dark variants with the OS-preference media query.
- `scheme: default` = light mode; `scheme: slate` = dark mode.
- `primary` controls the header/nav color; `accent` controls links and interactive elements.
- Material for MkDocs 9.x color names: red, pink, purple, deep purple, indigo, blue, light-blue, cyan, teal, green, light-green, lime, yellow, amber, orange, deep-orange, brown, grey, blue-grey, black, white.
- Custom colors can be set via CSS variables in `docs/assets/stylesheets/extra.css`:
  ```css
  :root {
    --md-primary-fg-color: #1b5e20;
    --md-accent-fg-color:  #76ff03;
  }
  ```
