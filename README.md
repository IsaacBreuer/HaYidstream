# HaYidstream

sharing my setup of streaming from HA online streams with tamplate selects 

I have only tested this on Google speakers and chromcast audio,  but should work on any speaker media player that allows steaming from a url.

the main setup use 2  tampalte selects one for listing all available speakers excluding the unavailable and the unknown, the dropdown options are displaying the friendly_name and the status of ranch media_player 

the second select is hardcoded for live streams available on yiddish24

the select itself acts as the automation 

the seleatct uses templates to set the stream url titles and artwork

we then  uses some Helper inputs to save some info 

for cards we use auto entity cards that dynamically shows all players playing 

custum cards I have used for this setup are all avaliable in hacs, you don't need the custom cards only if you use the card I have attached, but the core automations worked without it.


here are the list if cards used

+ auto-entities
+ layout-card
+ Mini Media Player
+ horizontal-layout


add 5 Helper entities,3 are used in the select to work,  the other 2 is just used for the card

+ input_text (text helper) named selectedstreamname
+ input_text (text helper) named selectedstreamurl
+ input_text (text helper) named selected_media_player
+ input_boolean.  (toggle helper) named playityes
+ then add a select (dropdown helper) named input_select.cover_art_option add the following options full-cover-fit, full-cover, cover,none

  the last select is just used in the card to specify how you want the cover art to appear on the mini player card





Here is the code used to setup the 2 tamplate selects, this goes into your configuration file..
You may want to spit your configuration file to keep it clean,  see homeassistant docs how to do it

```
template:
  - select:
     - name: "Available Media Players"
       state: "{{state_attr( states('input_text.selected_media_player'),'friendly_name') }} ~ {{states( states('input_text.selected_media_player') )}}"
       options: >
        {% set x = states.media_player
        | rejectattr('state','in',['unavailable','unknown'])
        | list%}
         {% set ns = namespace(result=[]) %}
        {% for s in x %}
        {% set ns.result= ns.result+[[s.name,s.state] | join (' ~ ')] %}
        {% endfor %}
        {{ns.result }}
       select_option:
         - variables:
             entity: >
                {{ states.media_player | selectattr('name', 'eq', option.split(' ~ ')[0])
                |map(attribute='entity_id')|join }}
         - service: input_text.set_value
           target:
             entity_id: input_text.selected_media_player 
           data:
              value: "{{ entity }}"
  - select:
     - name: "Select Stream"  
       options: "{{['אלגעמיינע קאלעקשאן','רואיגע מוזיק','נגינה אן מוזיק','חזנות קאלעקשאן','חתונה פרייליך קאלעקשאן','חתונה צווייטע טאנץ קאלעקשאן','חתונה קומזיץ קאלעקשאן','שבת קאלעקשאן','מוצאי שבת קאלעקשאן','חנוכה קאלעקשאן','פורים קאלעקשאן','פסח קאלעקשאן','סוכות','ימים נוראים','סאטמאר','מאטי אילאוויטש','באבוב','שבועות','מרדכי בן דוד','אברהם פריעד','ליפא שמעלצער','מיכאל שניצלער','משה גאלדמאן','אייזיק האניג','קינדער קווייער','לחיים','בעלזא','וויזניץ','סקולען','ל״ג בעומר','מוד׳זיץ','בנציון שענקער','אידישע ניגונים קאלעקשאן']}}"
       state: "{{ states('input_text.selectedstreamname') }}"
       select_option:
         - variables:
             mystream: >
                   {% set mapper = {
                    'אלגעמיינע קאלעקשאן' : '1',
                    'רואיגע מוזיק' : '11',
                    'נגינה אן מוזיק' : '3',
                    'חזנות קאלעקשאן' : '5',
                    'חתונה פרייליך קאלעקשאן' : '8',
                    'חתונה צווייטע טאנץ קאלעקשאן' : '10',
                    'חתונה קומזיץ קאלעקשאן' : '9',
                    'שבת קאלעקשאן' : '2',
                    'מוצאי שבת קאלעקשאן' : '6',
                    'חנוכה קאלעקשאן' : '4',
                    'פורים קאלעקשאן' : '12',
                    'פסח קאלעקשאן' : '13',
                    'סוכות' : '17',
                    'ימים נוראים' : '16',
                    'סאטמאר' : '35',
                    'מאטי אילאוויטש' : '34',
                    'באבוב' : '31',
                    'שבועות' : '19',
                    'מרדכי בן דוד' : '20',
                    'אברהם פריעד' : '21',
                    'ליפא שמעלצער' : '22',
                    'מיכאל שניצלער' : '23',
                    'משה גאלדמאן' : '24',
                    'אייזיק האניג' : '25',
                    'קינדער קווייער' : '26',
                    'לחיים' : '27',
                    'בעלזא' : '28',
                    'וויזניץ' : '29',
                    'סקולען' : '30',
                    'ל״ג בעומר' : '14',
                    'מוד׳זיץ' : '33',
                    'בנציון שענקער' : '32',
                    'אידישע ניגונים קאלעקשאן' : '7'
                     }%}
                     {{ mapper.get(option) }}
                     
                     
         - service: input_text.set_value
           target:
            entity_id: input_text.selectedstreamname 
           data:
              value: "{{ option }}"
         - service: input_text.set_value
           target:
            entity_id: input_text.selectedstreamurl 
           data:
              value: "{{ mystream }}"
         - choose:
              - conditions:
                  - " {{ states('input_boolean.playityes') =='on' }}"
                sequence: 
                    - variables:
                        mypicture: >
                            {% set mapper2 = {                   
                                '1' : 'https://cloudfront.yiddish24.com/%D7%90%D7%9C%D7%92%D7%A2%D7%9E%D7%99%D7%99%D7%A0%D7%A2-%D7%A7%D7%90%D7%9C%D7%A2%D7%A7%D7%A9%D7%90%D7%9F_1662090580.',
                                '11' : 'https://cloudfront.yiddish24.com/%D7%A8%D7%95%D7%90%D7%99%D7%92%D7%A2-%D7%9E%D7%95%D7%96%D7%99%D7%A7_1662090553.',
                                '3' : 'https://cloudfront.yiddish24.com/%D7%95%D7%95%D7%90%D7%A7%D7%90%D7%9C%D7%99%D7%A9_1662090512.',
                                '5' : 'https://cloudfront.yiddish24.com/%D7%97%D7%96%D7%A0%D7%95%D7%AA_1662090478.',
                                '8' : 'https://cloudfront.yiddish24.com/%D7%97%D7%AA%D7%95%D7%A0%D7%94-%D7%A4%D7%A8%D7%99%D7%99%D7%9C%D7%99%D7%9A_1662090439.',
                                '10' : 'https://cloudfront.yiddish24.com/%D7%A6%D7%95%D7%95%D7%99%D7%99%D7%98%D7%A2-%D7%98%D7%90%D7%A0%D7%A5_1662090413.',
                                '9' : 'https://cloudfront.yiddish24.com/%D7%A7%D7%95%D7%9E%D7%96%D7%99%D7%A5_1662096447.',
                                '2' : 'https://cloudfront.yiddish24.com/%D7%A9%D7%91%D7%AA_1662090342.',
                                '6' : 'https://cloudfront.yiddish24.com/%D7%9E%D7%95%D7%A6%D7%90%D7%99-%D7%A9%D7%91%D7%AA_1662090315.',
                                '4' : 'https://cloudfront.yiddish24.com/%D7%97%D7%A0%D7%95%D7%9B%D7%94_1662090285.',
                                '12' : 'https://cloudfront.yiddish24.com/%D7%A4%D7%95%D7%A8%D7%99%D7%9D_1662090243.',
                                '13' : 'https://cloudfront.yiddish24.com/pesach_1661965660.jpg',
                                '17' : 'https://cloudfront.yiddish24.com/%D7%A1%D7%95%D7%9B%D7%95%D7%AA_1662096684.',
                                '16' : 'https://cloudfront.yiddish24.com/%D7%99%D7%9E%D7%99%D7%9D-%D7%A0%D7%95%D7%A8%D7%90%D7%99%D7%9D_1662940956.',
                                '35' : 'https://cloudfront.yiddish24.com/%D7%A1%D7%90%D7%98%D7%9E%D7%90%D7%A8_1662941031.',
                                '34' : 'https://cloudfront.yiddish24.com/%D7%9E%D7%90%D7%98%D7%99-%D7%90%D7%99%D7%9C%D7%90%D7%95%D7%95%D7%99%D7%98%D7%A9_1662941151.',
                                '31' : 'https://cloudfront.yiddish24.com/%D7%91%D7%90%D7%91%D7%95%D7%91_1662941200.',
                                '19' : 'https://cloudfront.yiddish24.com/%D7%A9%D7%91%D7%95%D7%A2%D7%95%D7%AA_1662096087.',
                                '20' : 'https://cloudfront.yiddish24.com/%D7%9E%D7%A8%D7%93%D7%9B%D7%99-%D7%91%D7%9F-%D7%93%D7%95%D7%93_1662096107.jpg',
                                '21' : 'https://cloudfront.yiddish24.com/%D7%90%D7%91%D7%A8%D7%94%D7%9D-%D7%A4%D7%A8%D7%99%D7%A2%D7%93_1666716990.',
                                '22' : 'https://cloudfront.yiddish24.com/%D7%9C%D7%99%D7%A4%D7%90-%D7%A9%D7%9E%D7%A2%D7%9C%D7%A6%D7%A2%D7%A8_1666645561.',
                                '23' : 'https://cloudfront.yiddish24.com/%D7%9E%D7%99%D7%9B%D7%90%D7%9C-%D7%A9%D7%A0%D7%99%D7%A6%D7%9C%D7%A2%D7%A8_1662096167.',
                                '24' : 'https://cloudfront.yiddish24.com/%D7%9E%D7%A9%D7%94-%D7%92%D7%90%D7%9C%D7%93%D7%9E%D7%90%D7%9F_1662096198.',
                                '25' : 'https://cloudfront.yiddish24.com/%D7%90%D7%99%D7%99%D7%96%D7%99%D7%A7-%D7%94%D7%90%D7%A0%D7%99%D7%92_1662096204.',
                                '26' : 'https://cloudfront.yiddish24.com/%D7%A7%D7%99%D7%A0%D7%93%D7%A2%D7%A8-%D7%A7%D7%95%D7%95%D7%99%D7%99%D7%A2%D7%A8_1666645700.',
                                '27' : 'https://cloudfront.yiddish24.com/%D7%9C%D7%97%D7%99%D7%99%D7%9D_1662096216.',
                                '28' : 'https://cloudfront.yiddish24.com/%D7%91%D7%A2%D7%9C%D7%96%D7%90_1662096224.',
                                '29' : 'https://cloudfront.yiddish24.com/%D7%95%D7%95%D7%99%D7%96%D7%A0%D7%99%D7%A5_1662096229.',
                                '30' : 'https://cloudfront.yiddish24.com/%D7%A1%D7%A7%D7%95%D7%9C%D7%A2%D7%9F_1662096235.',
                                '14' : 'https://cloudfront.yiddish24.com/%D7%9C%D7%B4%D7%92-%D7%91%D7%A2%D7%95%D7%9E%D7%A8_1666645733.',
                                '33' : 'https://cloudfront.yiddish24.com/%D7%9E%D7%95%D7%93%D7%B3%D7%96%D7%99%D7%A5_1662941272.',
                                '32' : 'https://cloudfront.yiddish24.com/%D7%91%D7%A0%D7%A6%D7%99%D7%95%D7%9F-%D7%A9%D7%A2%D7%A0%D7%A7%D7%A2%D7%A8_1662941321.',
                                '7' : 'https://cloudfront.yiddish24.com/%D7%90%D7%99%D7%93%D7%99%D7%A9-%D7%A7%D7%90%D7%9C%D7%A2%D7%A7%D7%A9%D7%90%D7%9F_1666645673.'
                            }
                            %}

                                {{ mapper2.get(states("input_text.selectedstreamurl")) }}

                    - service: media_player.play_media
                      data:
                        media_content_id: 'https://music.yiddish24.com:5001/{{states("input_text.selectedstreamurl")}}'
                        media_content_type: Music
                        extra:
                         thumb: "{{mypicture}}"
                         title: "{{states('input_text.selectedstreamname')}}"
                      target:
                        entity_id: "{{ states('input_text.selected_media_player')}}"
```
note! if you already have something in your configuration under tamplate,  don't repeate the word tamplate,  Judy put the 2 selects under the parent tamplate node

check your configuration 
reload all yaml from developer tolls 

by now this should work, use the 2 selects to select  a player and a steam and if input_boolean.playityes is on it will play.


if you want my dynamic cards here is what i have used.

```
type: custom:layout-card
layout_type: custom:horizontal-layout
cards:
  - type: entities
    entities:
      - entity: select.available_media_players
        name: 'וועל אויס א ספיקער '
        icon: mdi:speaker-wireless
      - entity: select.select_stream
        name: וועל אויס מוזיק טשענעל
        icon: mdi:music
      - entity: input_boolean.playityes
        name: שפיל
      - entity: input_select.cover_art_option
    title: קול מבשר לייוו מוזיק
  - type: custom:auto-entities
    show_empty: true
    card:
      type: entities
      title: אלע שפילענדע ספיקערס
    filter:
      template: |-
        {% set speakers = states.media_player
          | rejectattr('state','in',['unavailable','unknown','off'])
          | list%}
        {% set coverartoption = states('input_select.cover_art_option') %}
        [ {% for speaker in expand(speakers)  %}


            {{
               {
                 'type': 'custom:mini-media-player',
                 'entity' : speaker.entity_id,
                 'artwork' : coverartoption,
                 'group': false,
                 'hide': {
                   'power': false,
                   'icon': true
                 }
               }
            }},
        {%- endfor %} ]
layout:
  margin: 0px 0px 0px 0px
  card_margin: 0
```

       
