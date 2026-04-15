# RKNHardering-defense

<img width="268" height="117" alt="RKNHardering-defense" src="https://github.com/user-attachments/assets/490609c1-82da-431e-87f6-c890f45c02cd" />

### Не проходим проверку [RKNHardering](https://github.com/xtclovver/RKNHardering) без рута, ~~тихо, не спеша, без суеты~~
<hr>

Итак, что же я хочу сказать:

***Скрыть наличие TUN-интерфейса и системного прокси на Android без root-прав архитектурно невозможно. Однако можно попытаться минимизировать утечку информации о подключении, например IP адреса сервера, что и предлагает данная инструкция.***

***Если у вас есть роутер Keenetic/Netcraze, можно настроить его, установив [SKeen](https://github.com/jinndi/SKeen). Примеры конфигураций находятся там же в папке `examples` (файлы `config-cheburnet*.json`). TUN не будет обнаруживаться, но это предназначено только для домашнего использования через Wi-Fi.***

1. Устанавливаем [S-UI](https://github.com/alireza0/s-ui) на сервер.

2. Настраиваем S-UI:
   - создаём клиента;
   - настраиваем подключения;
   - *(необязательно)* можно добавить подписку или ссылку прямо в клиенте.

3. Переходим в: **Настройки → JSON подписка → Редактор**

Добавляем следующий шаблон:

> ⚠️
> 
> Укажите свои доверенные приложения в поле `inbounds.tun.include_package` конфига ниже, имена можно узнать через приложение `Package Names Viewer` из `Google Play`.
> 
> Можно сделать наоборот: заменить `include_package` на `exclude_package`, в котором указать приложения, которым вы **не доверяете**, они исключатся из проксирования.
>
> В итоге должно быть что-то одно либо `include_package` либо `exclude_package`

> ⚠️ Если вы настраиваете всё это только для себя и не планируете давать ссылки на подписки кому-либо другому, кто не разбирается в этом, то просто настройте всё прямо в приложении sing-box: **Параметры → Перезапись профиля → Прокси для отдельных приложений**. В этом случае заполнять в шаблоне `include_package` и `exclude_package` не требуется - их необходимо удалить, чтобы не было конфликтов.


```json
{
  "log": {
    "disabled": false,
    "level": "debug",
    "timestamp": false
  },
  "dns": {
    "servers": [
      {
        "tag": "dns-hosts",
        "type": "hosts",
        "predefined": {
          "one.one.one.one": [
            "1.1.1.1",
            "1.0.0.1"
          ],
          "common.dot.dns.yandex.net": [
            "77.88.8.8",
            "77.88.8.1"
          ]
        }
      },
      {
        "type": "tls",
        "tag": "dns-proxy",
        "detour": "proxy",
        "domain_resolver": "dns-hosts",
        "server": "one.one.one.one"
      },
      {
        "type": "https",
        "tag": "dns-direct",
        "domain_resolver": "dns-hosts",
        "server": "common.dot.dns.yandex.net"
      },
      {
        "type": "fakeip",
        "tag": "fakeip",
        "inet4_range": "198.18.0.0/15"
      },
      {
        "type": "udp",
        "tag": "dns-resolver",
        "server": "77.88.8.8"
      }
    ],
    "rules": [
      {
        "action": "reject",
        "rule_set": "adguard",
        "method": "default"
      },
      {
        "action": "route",
        "rule_set": [
          "category-ip-geo-detect",
          "geosite-cheburnet",
          "geosite-category-ru"
        ],
        "server": "dns-direct"
      },
      {
        "action": "route",
        "type": "logical",
        "mode": "and",
        "rules": [
          {
            "domain_suffix": [
              ".lan",
              ".localdomain",
              ".example",
              ".invalid",
              ".localhost",
              ".test",
              ".local",
              ".home.arpa",
              ".msftconnecttest.com",
              ".msftncsi.com"
            ],
            "invert": true
          },
          {
            "query_type": [
              "A"
            ]
          }
        ],
        "client_subnet": "5.255.255.0/24",
        "server": "fakeip"
      }
    ],
    "disable_cache": false,
    "disable_expire": false,
    "independent_cache": true,
    "reverse_mapping": true,
    "final": "dns-proxy",
    "strategy": "ipv4_only"
  },
  "inbounds": [
    {
      "type": "tun",
      "address": "172.18.0.1/30",
      "mtu": 1500,
      "auto_route": true,
      "strict_route": true,
      "stack": "mixed",
      "udp_timeout": "3m",
      "endpoint_independent_nat": false,
      "include_package": [
        "ru.fourpda.client",
        "com.discord",
        "com.android.chrome",
        "org.mozilla.firefox",
        "com.brave.browser",
        "com.github.android",
        "com.openai.chatgpt",
        "com.instagram.android",
        "com.shazam.android",
        "com.soundcloud.android",
        "org.zwanoo.android.speedtest",
        "com.valvesoftware.android.steam.community",
        "org.telegram.messenger",
        "org.telegram.messenger.web",
        "com.instagram.barcelona",
        "com.ton_keeper",
        "tv.twitch.android.app",
        "com.twitter.android",
        "com.google.android.apps.docs",
        "com.google.android.apps.bard",
        "com.google.android.googlequicksearchbox",
        "com.google.android.webview",
        "com.google.android.gm",
        "com.android.vending",
        "com.google.android.apps.tachyon",
        "com.google.android.youtube",
        "anddea.youtube",
        "com.google.android.gms",
        "com.facebook.katana"
      ]
    }
  ],
  "auto_detect_interface": true,
  "final": "proxy",
  "default_domain_resolver": "dns-resolver",
  "find_process": true,
  "rules": [
    {
      "type": "logical",
      "mode": "or",
      "rules": [
        {
          "port": 53
        },
        {
          "protocol": "dns"
        }
      ],
      "action": "hijack-dns"
    },
    {
      "action": "reject",
      "protocol": "stun"
    },
    {
      "action": "route",
      "ip_is_private": true,
      "outbound": "direct"
    },
    {
      "action": "route",
      "rule_set": [
        "category-ip-geo-detect",
        "geosite-cheburnet",
        "geosite-category-ru"
      ],
      "outbound": "direct"
    },
    {
      "action": "resolve"
    },
    {
      "action": "route",
      "rule_set": "geoip-ru",
      "outbound": "direct"
    }
  ],
  "rule_set": [
    {
      "tag": "adguard",
      "type": "remote",
      "url": "https://github.com/jinndi/adguard-filter-list-srs/releases/latest/download/adguard-filter-list.srs",
      "format": "binary",
      "download_detour": "proxy"
    },
    {
      "tag": "category-ip-geo-detect",
      "type": "remote",
      "url": "https://github.com/KaringX/karing-ruleset/raw/sing/geo/geosite/category-ip-geo-detect.srs",
      "format": "binary",
      "download_detour": "proxy"
    },
    {
      "tag": "geosite-cheburnet",
      "type": "remote",
      "url": "https://github.com/jinndi/geosite-cheburnet/releases/latest/download/geosite-cheburnet.srs",
      "format": "binary",
      "download_detour": "proxy"
    },
    {
      "tag": "geoip-ru",
      "type": "remote",
      "url": "https://github.com/KaringX/karing-ruleset/raw/sing/geo/geoip/ru.srs",
      "format": "binary",
      "download_detour": "proxy"
    },
    {
      "tag": "geosite-category-ru",
      "type": "remote",
      "url": "https://github.com/KaringX/karing-ruleset/raw/sing/geo/geosite/category-ru.srs",
      "format": "binary",
      "download_detour": "proxy"
    }
  ],
  "experimental": {
    "cache_file": {
      "enabled": true,
      "path": "cache.db",
      "store_fakeip": true,
      "store_rdrc": true,
      "rdrc_timeout": "7d"
    }
  }
}
```

> Если возникают проблемы со звонками через прокси, удалите из секции `route.rules` следующий блок (действие выполняется на ваш страх и риск):

```json
    {
      "action": "reject",
      "protocol": "stun"
    },
```

4. Скачиваем из репозитория [sing-box](https://github.com/SagerNet/sing-box/releases/latest) последнюю стабильную APK-версию приложения для Android (пример:  
   `SFA-1.13.7-arm64-v8a.apk`) и устанавливаем!
   
5. Приложения, которым вы не доверяете, могут собирать сведения об установленных на вашем устройстве приложениях. Поэтому мы скрываем доверенные приложения, включая установленное приложение sing-box, с помощью настроек телефона. Например, на Realme это находится в разделе:

**Настройки → Безопасность и конфиденциальность → Скрыть приложения**

Все остальные VPN-приложения удаляем.

6. В созданном клиенте в S-UI нажимаем на значок QR-кода.  
   Откроется окно, где можно:
   - отсканировать ссылку на подписку через приложение sing-box  
   - или вручную скопировать и добавить ссылку на JSON-подписку

9. Подключаемся и радуемся, но не сильно.

~~Бит лютый вообще...~~

