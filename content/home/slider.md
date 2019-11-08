+++
# Slider widget.
widget = "slider"  # See https://sourcethemes.com/academic/docs/page-builder/
headless = true  # This file represents a page section.
active = true  # Activate this widget? true/false
weight = 1  # Order that this section will appear.

# Slide interval.
# Use `false` to disable animation or enter a time in ms, e.g. `5000` (5s).
interval = 5000

# Slide height (optional).
# E.g. `500px` for 500 pixels or `calc(100vh - 70px)` for full screen.
height = "300px"

# Slides.
# Duplicate an `[[item]]` block to add more slides.


[[item]]
  title = "Site under construction"
  content = "This website will be ready soon :smile:"
  align = "center"  # Choose `center`, `left`, or `right`.

  # Overlay a color or image (optional).
  #   Deactivate an option by commenting out the line, prefixing it with `#`.
  overlay_color = "#5d657a"  # An HTML color value.
  overlay_img = "slider_1.JPG"  # Image path relative to your `static/img/` folder.
  overlay_filter = 0.3  # Darken the image. Value in range 0-1.

  # Call to action button (optional).
  #   Activate the button by specifying a URL and button label below.
  #   Deactivate by commenting out parameters, prefixing lines with `#`.
  #cta_label = "Get Academic"
  #cta_url = "https://sourcethemes.com/academic/"
  #cta_icon_pack = "fas"
  #cta_icon = "graduation-cap"
  #use_pages = true
  #pages_title = "Biography"
  #pages_type = "about"
 
{{ \$ := .root }} {{ \$page := .page }} {{ \$hash\_id := .hash\_id }}

1.  

{{ range \$index, \$item := \$page.Params.item }} {{ \$query := where
site.RegularPages "Type" \$item.pages\_type }} {{\$recent\_item := index
(\$query) 0}} {{ \$i18n := "" }} {{ if eq \$item.pages\_type "post" }}
{{ \$i18n = "more\_posts" }} {{ else if eq \$item.pages\_type "talk" }}
{{ \$i18n = "more\_talks" }} {{ else if eq \$item.pages\_type
"publication" }} {{ \$i18n = "more\_publications" }} {{ else }} {{
\$i18n = "more\_pages" }} {{ end }}

{{ with \$item.title }}{{ . | markdownify | emojify }}{{ end }} {.hero-title}
===============================================================

{{ if \$item.use\_pages }}

{{ with \$item.pages\_title }}{{ . | markdownify | emojify }}{{ end }}
======================================================================

{{ with \$item.pages\_subtitle }}

{{ . | markdownify | emojify }}

{{ end }}

{{ partial "slider\_li" \$recent\_item }}

[{{ i18n \$i18n | default "See all" }} **]({{%20$item.pages_type%20}})

{{ end }} {{ with \$item.content }}

{{ . | markdownify | emojify }}

{{ end }} {{ if \$item.cta\_url }} {{ \$pack := or .cta\_icon\_pack
"fas" }} {{ \$pack\_prefix := \$pack }} {{ if in (slice "fab" "fas"
"far" "fal") \$pack }} {{ \$pack\_prefix = "fa" }} {{ end }}

[{{- with \$item.cta\_icon -}}**{{- end -}} {{- \$item.cta\_label |
emojify | safeHTML -}}]({{%20$item.cta_url%20}})

{{ end }}

{{ end }}


[[item]]
  title = ""
  content = ""
  align = "center"  # Choose `center`, `left`, or `right`.

  # Overlay a color or image (optional).
  #   Deactivate an option by commenting out the line, prefixing it with `#`.
  overlay_color = "#5d657a"  # An HTML color value.
  overlay_img = "header/koch.jpg"  # Image path relative to your `static/img/` folder.
  overlay_filter = 0.2  # Darken the image. Value in range 0-1.

  # Call to action button (optional).
  #   Activate the button by specifying a URL and button label below.
  #   Deactivate by commenting out parameters, prefixing lines with `#`.
  cta_label = "Check out my recent post"
  cta_url = "https://cahuparo.me/post/"
  cta_icon_pack = "fas"
  cta_icon = "graduation-cap"
  
  use_pages = true
  pages_title = "Recent posts"
  pages_type = "post"

#[[item]]
#  title = "Left"
#  content = "I am left aligned :smile:"
#  align = "left"

#  overlay_color = "#555"  # An HTML color value.
#  overlay_img = ""  # Image path relative to your `static/img/` folder.
#  overlay_filter = 0.5  # Darken the image. Value in range 0-1.

#[[item]]
#  title = "Right"
#  content = "I am right aligned :smile:"
#  align = "right"

#  overlay_color = "#333"  # An HTML color value.
#  overlay_img = ""  # Image path relative to your `static/img/` folder.
#  overlay_filter = 0.5  # Darken the image. Value in range 0-1.
+++
