# HaYidstream
Update 4, updated URl , ADDED new Stream, updated Photos


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
       #options: "{{['אלגעמיינע קאלעקשאן','רואיגע מוזיק','נגינה אן מוזיק','חזנות קאלעקשאן','חתונה פרייליך קאלעקשאן','חתונה צווייטע טאנץ קאלעקשאן','חתונה קומזיץ קאלעקשאן','שבת קאלעקשאן','מוצאי שבת קאלעקשאן','חנוכה קאלעקשאן','פורים קאלעקשאן','פסח קאלעקשאן','סוכות','ימים נוראים','סאטמאר','מאטי אילאוויטש','באבוב','שבועות','מרדכי בן דוד','אברהם פריעד','ליפא שמעלצער','מיכאל שניצלער','משה גאלדמאן','אייזיק האניג','קינדער קווייער','לחיים','בעלזא','וויזניץ','סקולען','ל״ג בעומר','מוד׳זיץ','בנציון שענקער','אידישע ניגונים קאלעקשאן','שירה מיקס','שירה פרייליך','שירה רילעקס','שירה וואקאל','שירה שבת','שירה פסח','שירה ימים נוראים']}}"
       options: "{{['אלגעמיינע קאלעקשאן','רואיגע מוזיק','נגינה אן מוזיק','חזנות קאלעקשאן','חתונה פרייליך קאלעקשאן','חתונה צווייטע טאנץ קאלעקשאן','קומזיץ קאלעקשאן','שבת קאלעקשאן','מוצאי שבת קאלעקשאן','חנוכה קאלעקשאן','פורים קאלעקשאן','לעבעדיג קאלעקשאן','פסח קאלעקשאן','סוכות','ימים נוראים','פיאנע','מעדיטעשאן','סאטמאר','חבד','מאטי אילאוויטש','אביש בראדט','באבוב','שבועות','מרדכי בן דוד','אברהם פריעד','ליפא שמעלצער','מיכאל שניצלער','אהרלה סאמעט','דודי קאליש','זאנוויל וויינברגר','משה גאלדמאן','אייזיק האניג','קינדער קווייער','לחיים','בעלזא','וויזניץ','סקולען','ל״ג בעומר','מוד׳זיץ','בנציון שענקער','אידישע ניגונים קאלעקשאן','בערי וועבער','ברוך לוין','ישראל ווערדיגער','שלומי גרטנר','שמחה ליינר','יהודה גרין','יעקב דאסקאל','מוטי שטיינמץ','שלומי דאסקאל',' שירה מיקס','שירה פרייליך','שירה רילעקס','שירה וואקאל','שירה שבת','שירה פסח','שירה ימים נוראים']}}"
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
                    'קומזיץ קאלעקשאן':'9^yid',
                    'שבת קאלעקשאן':'2^yid',
                    'מוצאי שבת קאלעקשאן':'6^yid',
                    'חנוכה קאלעקשאן':'4^yid',
                    'פורים קאלעקשאן':'12^yid',
                    'לעבעדיג קאלעקשאן':'37^yid',
                    'פסח קאלעקשאן':'13^yid',
                    'סוכות':'17^yid',
                    'ימים נוראים':'16^yid',
                    'פיאנע':'38^yid',
                    'מעדיטעשאן':'39^yid',
                    'סאטמאר':'35^yid',
                    'חבד':'40^yid',
                    'מאטי אילאוויטש':'34^yid',
                    'אביש בראדט':'41^yid',
                    'באבוב':'31^yid',
                    'שבועות':'19^yid',
                    'מרדכי בן דוד':'20^yid',
                    'אברהם פריעד':'21^yid',
                    'ליפא שמעלצער':'22^yid',
                    'מיכאל שניצלער':'23^yid',
                    'אהרלה סאמעט':'47^yid',
                    'דודי קאליש':'48^yid',
                    'זאנוויל וויינברגר':'53^yid',
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
                    'בערי וועבער':'42^yid',
                    'ברוך לוין':'43^yid',
                    'ישראל ווערדיגער':'44^yid',
                    'שלומי גרטנר':'45^yid',
                    'שמחה ליינר':'46^yid',
                    'יהודה גרין':'49^yid',
                    'יעקב דאסקאל':'50^yid',
                    'מוטי שטיינמץ':'51^yid',
                    'שלומי דאסקאל':'52^yid',
                    'שירה מיקס':'10^shi',
                    'שירה פרייליך':'2^shi',
                    'שירה רילעקס':'14^shi',
                    'שירה וואקאל':'8^shi',
                    'שירה שבת':'3^shi',
                    'שירה פסח':'13^shi',
                    'שירה ימים נוראים':'7^shi'
                     }%}
                    {% set mapper222 =  mapper | list %}
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
                                '1^yid' : 'https://i.ibb.co/fv2k7HM/all.jpg',
                                '11^yid' : 'https://i.ibb.co/nj4rYGw/slow.jpg',
                                '3^yid' : 'https://i.ibb.co/xs9RtmD/vocal.jpg',
                                '5^yid' : 'https://i.ibb.co/FmDw2wX/chazunas.jpg',
                                '8^yid' : 'https://i.ibb.co/rb4y1s5/chasunah.jpg',
                                '10^yid' : 'https://i.ibb.co/DGqK7dC/secunddence.jpg',
                                '9^yid' : 'https://i.ibb.co/Z83GfNh/kumzitz.jpg',
                                '2^yid' : 'https://i.ibb.co/KKJJSPj/shabos.jpg',
                                '6^yid' : 'https://i.ibb.co/9VmWbkW/hadalah.jpg',
                                '4^yid' : 'https://i.ibb.co/LnPPzrv/chanukah.jpg',
                                '12^yid' : 'https://i.ibb.co/5vzqqTj/purimn.jpg',
                                '37^yid' : 'https://i.ibb.co/jVqXK56/happy.jpg',
                                '13^yid' : 'https://i.ibb.co/VqJ3t9P/paisach.jpg',
                                '17^yid' : 'https://i.ibb.co/KDpQbgb/suckas.jpg',
                                '16^yid' : 'https://i.ibb.co/19QyC06/yomimnoiruim.png',
                                '38^yid' : 'https://i.ibb.co/0K1qk3k/piano.jpg',
                                '39^yid' : 'https://i.ibb.co/c2pGc25/maditation.png',
                                '35^yid' : 'https://i.ibb.co/w4MgNT6/satmar.jpg',
                                '40^yid' : 'https://i.ibb.co/cXrCRrT/chabad.png',
                                '34^yid' : 'https://i.ibb.co/y8jC2Fm/ilowitz.jpg',
                                '41^yid' : 'https://i.ibb.co/CP0rXR9/Abish-Brodt.jpg',
                                '31^yid' : 'https://i.ibb.co/JqzR55s/bobov.jpg',
                                '19^yid' : 'https://i.ibb.co/r516xF6/shvius.jpg',
                                '20^yid' : 'https://i.ibb.co/1KmCZLN/mbd.jpg',
                                '21^yid' : 'https://i.ibb.co/9HwmK3c/avrumfried.jpg',
                                '22^yid' : 'https://i.ibb.co/jyqgHVQ/schmeltzer.jpg',
                                '23^yid' : 'https://i.ibb.co/SwbxGRV/schnitzler.jpg',
                                '47^yid' : 'https://i.ibb.co/4f9mhx7/ahrla-samet.jpg',
                                '48^yid' : 'https://i.ibb.co/nf6r7N3/Dudi-Kalish.png',
                                '53^yid' : 'https://i.ibb.co/SVmGN8k/wienberger.png',
                                '24^yid' : 'https://i.ibb.co/YZ1nhzb/Goldman.png',
                                '25^yid' : 'https://i.ibb.co/NCtxHSm/isaac-Honig.jpg',
                                '26^yid' : 'https://i.ibb.co/p6HfBqn/yingerlich.png',
                                '27^yid' : 'https://i.ibb.co/tJxcxr5/lchaim.jpg',
                                '28^yid' : 'https://i.ibb.co/chNhXc7/belz.jpg',
                                '29^yid' : 'https://i.ibb.co/C1JFsHq/viznizt.jpg',
                                '30^yid' : 'https://i.ibb.co/1QJM7nF/skulen.png',
                                '14^yid' : 'https://i.ibb.co/2jzQDsr/lagboimer.jpg',
                                '33^yid' : 'https://i.ibb.co/NLDbrNd/mudzitz.png',
                                '32^yid' : 'https://i.ibb.co/qWjkPQv/shenker.jpg',
                                '7^yid' : 'https://i.ibb.co/XDGNsxt/yiddish.jpg',
                                '42^yid' : 'https://i.ibb.co/Z1L38dn/berry-Webber.jpg',
                                '43^yid' : 'https://i.ibb.co/GTTnKTf/burachlevin.jpg',
                                '44^yid' : 'https://i.ibb.co/vqPCZZZ/verdiger.jpg',
                                '45^yid' : 'https://i.ibb.co/s1xM7qw/gertner.jpg',
                                '46^yid' : 'https://i.ibb.co/RBYwvD3/leiner.jpg',
                                '49^yid' : 'https://i.ibb.co/zZb3MBD/green.jpg',
                                '50^yid' : 'https://i.ibb.co/JsrJWyv/daskel-Yanky.png',
                                '51^yid' : 'https://i.ibb.co/qyvvJdT/mottysteinment.jpg',
                                '52^yid' : 'https://i.ibb.co/qFp0BF1/Daskal.jpg',
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
                       # media_content_id: '{{ "https://music.yiddish24.com" if  (states("input_text.selectedstreamurl")).split("^")[1] == "yid" else "https://music.shira24.com"}}:5001/{{states("input_text.selectedstreamurl").split("^")[0]}}'
                        media_content_id: '{{ "https://music.y24.app" if  (states("input_text.selectedstreamurl")).split("^")[1] == "yid" else "https://music.shira24.com:5001"}}/{{states("input_text.selectedstreamurl").split("^")[0]}}'
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
        name: וועל אויס א ספיקער
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
          | selectattr('state','in',['playing','paused'])
          | rejectattr('entity_id','in',integration_entities('Music Assistant'))
          | selectattr('attributes.media_content_id', 'defined') 
          | selectattr('attributes.media_content_id','contains', 'https') 
          | list%}
          {% set playertouse = 'custom:hui-media-control-card' if states('input_select.cover_art_option')=="Built-in player" else 'custom:mini-media-player' %}

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

       
