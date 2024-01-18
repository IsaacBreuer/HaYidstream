# HaYidstream
Update 3 added shia 24  Just note shira server is very slow so it will buffer like 10 seconds before starting the stream


sharing my setup of streaming from HA online streams with tamplate selects 

I have only tested this on Google speakers and chromcast audio,  but should work on any speaker media player that allows steaming from a url.

the main setup use 2  tampalte selects one for listing all available speakers excluding the unavailable and the unknown, the dropdown options are displaying the friendly_name and the status of each media_player 


the second select is hardcoded for live streams available on yiddish24

the select itself acts as the automation 

the select uses templates to set the stream url titles and artwork

we then  uses some Helper inputs to save some info 

for cards we use auto entity cards that dynamically shows all players playing.

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
+ then add a input_select (dropdown helper) named cover_art_option add the following options full-cover-fit, full-cover, cover, icon, Built-in player

  the last select is just used in the card to specify how you want the cover art to appear on the mini player card

`	note changes to of options used in input_select.cover_art_option from previus versions `	


Here is the code used to setup the 2 tamplate selects, this goes into your configuration file..
You may want to spit your configuration file to keep it clean,  see homeassistant docs how to do it

```
template:
  - select:
     - name: "Available Media Players"
       state: "{{state_attr( states('input_text.selected_media_player'),'friendly_name') }} ~ {{states( states('input_text.selected_media_player') )}}"
       options: >
        {% set x = states.media_player
        |rejectattr('entity_id', 'in',integration_entities('Music Assistant'))
        | rejectattr('state','in',['unavailable','unknown'])
        | list%}
         {% set ns = namespace(result=[]) %}
        {% for s in x %}
        
        {%  if  s.attributes.supported_features|bitwise_and(512) %}
        {% set ns.result= ns.result+[[s.name,s.state] | join (' ~ ')] %}
        {% endif %}
  
  
        
        {% endfor %}
        {{ns.result }}
       select_option:
         - variables:
             entity: >
                 {{ states.media_player
                 |rejectattr('entity_id', 'in',integration_entities('Music Assistant'))
                 | rejectattr('state','in',['unavailable','unknown'])
                 | selectattr('name', 'eq', option.split(' ~ ')[0])
                 |map(attribute='entity_id')|first }}
         - service: input_text.set_value
           target:
             entity_id: input_text.selected_media_player 
           data:
              value: "{{ entity }}"
  - select:
     - name: "Select Stream"  
       options: "{{['אלגעמיינע קאלעקשאן','רואיגע מוזיק','נגינה אן מוזיק','חזנות קאלעקשאן','חתונה פרייליך קאלעקשאן','חתונה צווייטע טאנץ קאלעקשאן','חתונה קומזיץ קאלעקשאן','שבת קאלעקשאן','מוצאי שבת קאלעקשאן','חנוכה קאלעקשאן','פורים קאלעקשאן','פסח קאלעקשאן','סוכות','ימים נוראים','סאטמאר','מאטי אילאוויטש','באבוב','שבועות','מרדכי בן דוד','אברהם פריעד','ליפא שמעלצער','מיכאל שניצלער','משה גאלדמאן','אייזיק האניג','קינדער קווייער','לחיים','בעלזא','וויזניץ','סקולען','ל״ג בעומר','מוד׳זיץ','בנציון שענקער','אידישע ניגונים קאלעקשאן','שירה מיקס','שירה פרייליך','שירה רילעקס','שירה וואקאל','שירה שבת','שירה פסח','שירה ימים נוראים']}}"
       state: "{{ states('input_text.selectedstreamname') }}"
       select_option:
         - variables:
             mystream: >
                   {% set mapper = {
                    'אלגעמיינע קאלעקשאן':'1^yid',
                    'רואיגע מוזיק':'11^yid',
                    'נגינה אן מוזיק':'3^yid',
                    'חזנות קאלעקשאן':'5^yid',
                    'חתונה פרייליך קאלעקשאן':'8^yid',
                    'חתונה צווייטע טאנץ קאלעקשאן':'10^yid',
                    'חתונה קומזיץ קאלעקשאן':'9^yid',
                    'שבת קאלעקשאן':'2^yid',
                    'מוצאי שבת קאלעקשאן':'6^yid',
                    'חנוכה קאלעקשאן':'4^yid',
                    'פורים קאלעקשאן':'12^yid',
                    'פסח קאלעקשאן':'13^yid',
                    'סוכות':'17^yid',
                    'ימים נוראים':'16^yid',
                    'סאטמאר':'35^yid',
                    'מאטי אילאוויטש':'34^yid',
                    'באבוב':'31^yid',
                    'שבועות':'19^yid',
                    'מרדכי בן דוד':'20^yid',
                    'אברהם פריעד':'21^yid',
                    'ליפא שמעלצער':'22^yid',
                    'מיכאל שניצלער':'23^yid',
                    'משה גאלדמאן':'24^yid',
                    'אייזיק האניג':'25^yid',
                    'קינדער קווייער':'26^yid',
                    'לחיים':'27^yid',
                    'בעלזא':'28^yid',
                    'וויזניץ':'29^yid',
                    'סקולען':'30^yid',
                    'ל״ג בעומר':'14^yid',
                    'מוד׳זיץ':'33^yid',
                    'בנציון שענקער':'32^yid',
                    'אידישע ניגונים קאלעקשאן':'7^yid',
                    'שירה מיקס':'10^shi',
                    'שירה פרייליך':'2^shi',
                    'שירה רילעקס':'14^shi',
                    'שירה וואקאל':'8^shi',
                    'שירה שבת':'3^shi',
                    'שירה פסח':'13^shi',
                    'שירה ימים נוראים':'7^shi'
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
                                '1^yid' : 'https://cloudfront.yiddish24.com/%D7%90%D7%9C%D7%92%D7%A2%D7%9E%D7%99%D7%99%D7%A0%D7%A2-%D7%A7%D7%90%D7%9C%D7%A2%D7%A7%D7%A9%D7%90%D7%9F_1662090580.',
                                '11^yid' : 'https://cloudfront.yiddish24.com/%D7%A8%D7%95%D7%90%D7%99%D7%92%D7%A2-%D7%9E%D7%95%D7%96%D7%99%D7%A7_1662090553.',
                                '3^yid' : 'https://cloudfront.yiddish24.com/%D7%95%D7%95%D7%90%D7%A7%D7%90%D7%9C%D7%99%D7%A9_1662090512.',
                                '5^yid' : 'https://cloudfront.yiddish24.com/%D7%97%D7%96%D7%A0%D7%95%D7%AA_1662090478.',
                                '8^yid' : 'https://cloudfront.yiddish24.com/%D7%97%D7%AA%D7%95%D7%A0%D7%94-%D7%A4%D7%A8%D7%99%D7%99%D7%9C%D7%99%D7%9A_1662090439.',
                                '10^yid' : 'https://cloudfront.yiddish24.com/%D7%A6%D7%95%D7%95%D7%99%D7%99%D7%98%D7%A2-%D7%98%D7%90%D7%A0%D7%A5_1662090413.',
                                '9^yid' : 'https://cloudfront.yiddish24.com/%D7%A7%D7%95%D7%9E%D7%96%D7%99%D7%A5_1662096447.',
                                '2^yid' : 'https://cloudfront.yiddish24.com/%D7%A9%D7%91%D7%AA_1662090342.',
                                '6^yid' : 'https://cloudfront.yiddish24.com/%D7%9E%D7%95%D7%A6%D7%90%D7%99-%D7%A9%D7%91%D7%AA_1662090315.',
                                '4^yid' : 'https://cloudfront.yiddish24.com/%D7%97%D7%A0%D7%95%D7%9B%D7%94_1662090285.',
                                '12^yid' : 'https://cloudfront.yiddish24.com/%D7%A4%D7%95%D7%A8%D7%99%D7%9D_1662090243.',
                                '13^yid' : 'https://cloudfront.yiddish24.com/pesach_1661965660.jpg',
                                '17^yid' : 'https://cloudfront.yiddish24.com/%D7%A1%D7%95%D7%9B%D7%95%D7%AA_1662096684.',
                                '16^yid' : 'https://cloudfront.yiddish24.com/%D7%99%D7%9E%D7%99%D7%9D-%D7%A0%D7%95%D7%A8%D7%90%D7%99%D7%9D_1662940956.',
                                '35^yid' : 'https://cloudfront.yiddish24.com/%D7%A1%D7%90%D7%98%D7%9E%D7%90%D7%A8_1662941031.',
                                '34^yid' : 'https://cloudfront.yiddish24.com/%D7%9E%D7%90%D7%98%D7%99-%D7%90%D7%99%D7%9C%D7%90%D7%95%D7%95%D7%99%D7%98%D7%A9_1662941151.',
                                '31^yid' : 'https://cloudfront.yiddish24.com/%D7%91%D7%90%D7%91%D7%95%D7%91_1662941200.',
                                '19^yid' : 'https://cloudfront.yiddish24.com/%D7%A9%D7%91%D7%95%D7%A2%D7%95%D7%AA_1662096087.',
                                '20^yid' : 'https://cloudfront.yiddish24.com/%D7%9E%D7%A8%D7%93%D7%9B%D7%99-%D7%91%D7%9F-%D7%93%D7%95%D7%93_1662096107.jpg',
                                '21^yid' : 'https://cloudfront.yiddish24.com/%D7%90%D7%91%D7%A8%D7%94%D7%9D-%D7%A4%D7%A8%D7%99%D7%A2%D7%93_1666716990.',
                                '22^yid' : 'https://cloudfront.yiddish24.com/%D7%9C%D7%99%D7%A4%D7%90-%D7%A9%D7%9E%D7%A2%D7%9C%D7%A6%D7%A2%D7%A8_1666645561.',
                                '23^yid' : 'https://cloudfront.yiddish24.com/%D7%9E%D7%99%D7%9B%D7%90%D7%9C-%D7%A9%D7%A0%D7%99%D7%A6%D7%9C%D7%A2%D7%A8_1662096167.',
                                '24^yid' : 'https://cloudfront.yiddish24.com/%D7%9E%D7%A9%D7%94-%D7%92%D7%90%D7%9C%D7%93%D7%9E%D7%90%D7%9F_1662096198.',
                                '25^yid' : 'https://cloudfront.yiddish24.com/%D7%90%D7%99%D7%99%D7%96%D7%99%D7%A7-%D7%94%D7%90%D7%A0%D7%99%D7%92_1662096204.',
                                '26^yid' : 'https://cloudfront.yiddish24.com/%D7%A7%D7%99%D7%A0%D7%93%D7%A2%D7%A8-%D7%A7%D7%95%D7%95%D7%99%D7%99%D7%A2%D7%A8_1666645700.',
                                '27^yid' : 'https://cloudfront.yiddish24.com/%D7%9C%D7%97%D7%99%D7%99%D7%9D_1662096216.',
                                '28^yid' : 'https://cloudfront.yiddish24.com/%D7%91%D7%A2%D7%9C%D7%96%D7%90_1662096224.',
                                '29^yid' : 'https://cloudfront.yiddish24.com/%D7%95%D7%95%D7%99%D7%96%D7%A0%D7%99%D7%A5_1662096229.',
                                '30^yid' : 'https://cloudfront.yiddish24.com/%D7%A1%D7%A7%D7%95%D7%9C%D7%A2%D7%9F_1662096235.',
                                '14^yid' : 'https://cloudfront.yiddish24.com/%D7%9C%D7%B4%D7%92-%D7%91%D7%A2%D7%95%D7%9E%D7%A8_1666645733.',
                                '33^yid' : 'https://cloudfront.yiddish24.com/%D7%9E%D7%95%D7%93%D7%B3%D7%96%D7%99%D7%A5_1662941272.',
                                '32^yid' : 'https://cloudfront.yiddish24.com/%D7%91%D7%A0%D7%A6%D7%99%D7%95%D7%9F-%D7%A9%D7%A2%D7%A0%D7%A7%D7%A2%D7%A8_1662941321.',
                                '7^yid'  : 'https://cloudfront.yiddish24.com/%D7%90%D7%99%D7%93%D7%99%D7%A9-%D7%A7%D7%90%D7%9C%D7%A2%D7%A7%D7%A9%D7%90%D7%9F_1666645673.',
                                '10^shi' :'https://shira24.com/img/shira-logo.png',
                                '2^shi' :'https://shira24.com/img/shira-logo.png',
                                '14^shi' :'https://shira24.com/img/shira-logo.png',
                                '8^shi' :'https://shira24.com/img/shira-logo.png',
                                '3^shi' :'https://shira24.com/img/shira-logo.png',
                                '13^shi' :'https://shira24.com/img/shira-logo.png',
                                '7^shi' :'https://shira24.com/img/shira-logo.png'

                            }
                            %}

                                {{ mapper2.get(states("input_text.selectedstreamurl")) }}

                    - service: media_player.play_media
                      data:
                        media_content_id: '{{ "https://music.yiddish24.com" if  (states("input_text.selectedstreamurl")).split("^")[1] == "yid" else "https://music.shira24.com"}}:5001/{{states("input_text.selectedstreamurl").split("^")[0]}}'
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
          | selectattr('state','in',['playing','pjaused'])
          | list%}
          {% set playertouse = 'custom:mini-media-player' %}
          {% if states('input_select.cover_art_option')=="Built-in player" %}
          {% set playertouse = 'custom:hui-media-control-card' %}
          {%endif%}
           


        [ {% for speaker in expand(speakers)  %}
        {% if states('input_select.cover_art_option')=="icon" %}

            {{
               {
                 'type': playertouse,
                 'entity' : speaker.entity_id,
                 'group': false,
                 'hide': { 
                   'power': false,
                   'icon': false
                 }
               }
            }},
        {%else%}
            {{
               {
                 'type': playertouse,
                 'entity' : speaker.entity_id,
                 'artwork' : states('input_select.cover_art_option'),
                 'group': false,
                 'hide': { 
                   'power': false,
                   'icon': not states('input_select.cover_art_option')=="none"
                 }
               }
            }},

         {% endif %}   
        {%- endfor %} ]
layout:
  margin: 0px 0px 0px 0px
  card_margin: 0

```
# Screenshots 


## 1 payer on full-cover

![alt text](https://imgpile.com/images/GEFwMx.jpg)



## 2 player playing , cover 

![alt text](https://imgpile.com/images/GEFUIL.jpg)

       
