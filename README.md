# lovelace-gen.py A lovelace configuration generator for [homeassistant](https://www.home-assistant.io)

This script will generate a `ui-lovelace.yaml` file based off a file `lovelace/main.yaml`. Other files in the `lovelace` directory can be included into the configuration using the `!include filename` statement. This allows for separation of the configuration into several files, and reuse of cards.

## Usage

Create a directory `<homeassistant config dir>/lovelace` and the file `<homeassistant config dir>/lovelace/main.yaml`.

Inside your homeassistant config directory, run the command:

    lovelace-gen.py

This will create the file `ui_lovelace.yaml`.

#### Usage in Hass.io

Create a directory `config/lovelace` and the file `config/lovelace/main.yaml`.

In your configuration.yaml file, make a shell command:

```yaml
shell_command:
  lovelace_gen: 'python /config/lovelace-gen.py'
  ```

Restart Home Assistant. Then run the service `shell_command.lovelace_gen`, preferably from `<hass_ip_address:port>/dev-service`.

This will create the file `ui_lovelace.yaml`.

## Special commands

The following commands can be used in `lovelace/main.yaml` or any file included using the `!include` command.

- `!include <filename>` is replaced with the contents of `lovelace/<filename>`.
- `!resource [<path>/]<filename>` will copy the file `lovelace/<path>/<filename>` to `www/lovelace/<filename>` and be replaced with `/local/lovelace/<filename>`. A timestamp will be added after the url to make sure any cache of the file is invalidated between runs.
- [jinja2 templates](http://jinja.pocoo.org/docs/2.10/templates/) allows for variables, loops, macros and flow controll.


## Example

`lovelace/main.yaml`:

```yaml
title: My Awesome Home

# Copy resources from anywhere to www/lovelace and include them
resources:
  - url: !resource monster-card/monster-card.js
    type: js
  - url: !resource /home/hass/tracker-card/tracker-card.js?v=0.1.4
    type: js

views:
  - title: Home
    id: home
    cards:
      - type: picture_entity
        image: http://placekitten.com/6/200/300
        entity: light.cat_light
      - !include presence_tracker.yaml
      - !include lights.yaml
  - !include family_view.yaml
```

---

`lovelace/family_view.yaml`:

```yaml
title: Family
id: family
cards:
  - !include presence_tracker.yaml
```

---

`lovelace/presence_tracker.yaml`:

```yaml
type: entity_filter
entities:
  - device_tracker.paulus
  - device_tracker.anne_there
state_filter:
  - 'home'
card:
  type: glance
  title: People that are home
```

---

`lovelace/lights.yaml`

```yaml
{% macro light_pe(switch, image) %}
{ type: picture-entity,
  entity: {{switch}},
  image: !resource {{image}}
  show_state: false,
  tap_action: toggle
}
{% endmacro %}
type: vertical-stack
cards:
  - {{ light_pe('light.kitchen_light', 'images/kitchen_light.png') }}
  - {{ light_pe('light.bedroom', 'images/bedroom_light.png') }}
```

---

Generated `ui-lovelace.yaml`:

```yaml

# This file is automatically generated by lovelace-gen.py
# https://github.com/thomasloven/homeassistant-lovelace-gen
# Any changes made to it will be overwritten the next time the script is run.

title: My Awesome Home
resources:
- {type: js, url: /local/lovelace/monster-card.js?1533670932.854793}
- {type:js, url: /local/lovelace/tracker-card.js?1533670932.854793&v=0.1.4}
views:
- cards:
  - {entity: light.cat_light, image: 'http://placekitten.com/6/200/300', type: picture_entity}
  - card: {title: People that are home, type: glance}
    entities: [device_tracker.paulus, device_tracker.anne_there]
    state_filter: [home]
    type: entity_filter
  - cards:
      - {type: picture_entity, entity: light.kitchen_light, image: /local/lovelace/kitchen_light.png, show_state: false, tap_action: toggle}
      - {type: picture_entity, entity: light.bedroom, image: /local/lovelace/bedroom_light.png, show_state: false, tap_action: toggle}
    type: vertical-stack
  id: home
  title: Home
- cards:
  - card: {title: People that are home, type: glance}
    entities: [device_tracker.paulus, device_tracker.anne_there]
    state_filter: [home]
    type: entity_filter
  id: family
  title: Family
```
