# Инструкция прохождения проверки в RKNHardering на основе панели Sub-Store

1. Устанавливаем [Sub-Store](https://github.com/jinndi/Sub-Store-Docker) на сервер.

2. Настраиваем `Sub-Store`:

  - **`Profile`**: Во-первых, переключитесь на английский язык (в меню слева самая последняя вкладка - `Profile`, там вверху справа будет значок переключения).
  
  - **`Subs`**: Создаём подписку (первая вкладка - `Subs`). Доступно добавление по ссылке (`Remote URL`) либо локальная вставка (`Local`), можно использовать и то и другое сразу. В поле `Name` вводим название на английском - допустим, `mysub`. Запомним его и сохраним настройки кнопкой внизу, завершив добавление подписки.

  - **`File` (`Source`)**: Переходим во вкладку `File`, нажимаем сверху на `+` и выбираем в появившемся боковом меню `File`. Задаём имя на английском в поле `Name` (например, `myfile`). Ниже убеждаемся, что в поле `Type` выбрано значение `File`, а не `Mihomo Profile`. Затем в поле `Source` переключаемся в режим `Local`.

Добавляем следующий шаблон в поле ввода:

> ⚠️
> 
> Укажите свои доверенные приложения в поле `inbounds.tun.include_package` конфига ниже, имена можно узнать через приложение `Package Names Viewer` из `Google Play`.
> 
> Можно сделать наоборот: заменить `include_package` на `exclude_package`, в котором указать приложения, которым вы **не доверяете**, они исключатся из проксирования.
>
> ⚠️ В итоге **должно использоваться что-то одно** — либо `include_package`, либо `exclude_package`.
Однако в приоритете лучше использовать `include_package`, так как если вы будете делиться ссылкой на подписку с кем-то ещё, невозможно заранее узнать, какое потенциально вредное приложение может появиться на его устройстве и которое потребуется исключить.

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
          "one.one.one.one": ["1.1.1.1", "1.0.0.1"],
          "dns.yandex.net": ["77.88.8.8", "77.88.8.1"]
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
        "server": "dns.yandex.net"
      },
      {
        "tag": "fakeip",
        "type": "fakeip",
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
        "rule_set": "adguard" 
      },
      {
        "clash_mode": "Direct",
        "server": "dns-direct"
      },
      {
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
            "rule_set": "fakeip-filter",
            "invert": true
          },
          {
            "query_type": "A"
          }
        ],
        "client_subnet": "5.255.255.0/24",
        "server": "fakeip"
      },
      {
        "clash_mode": "Global",
        "server": "dns-proxy"
      }
    ],
    "independent_cache": true,
    "reverse_mapping": true,
    "final": "dns-proxy",
    "strategy": "ipv4_only"
  },
  "inbounds": [
    {
      "type": "tun",
      "address": [
        "172.18.0.1/30",
        "fdfe:dcba:9876::1/126"
      ],
      "mtu": 1500,
      "auto_route": true,
      "strict_route": true,
      "stack": "mixed",
      "udp_timeout": "1m",
      "endpoint_independent_nat": false
    }
  ],
  "route": {
    "auto_detect_interface": true,
    "override_android_vpn": true,
    "final": "proxy",
    "default_domain_resolver": "dns-resolver",
    "find_process": true,
    "rules": [
      {
        "network": "icmp",
        "outbound": "direct"
      },
      {
        "action": "sniff"
      },
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
        "protocol": "stun",
        "action": "reject"
      },
      {
        "ip_is_private": true,
        "outbound": "direct"
      },
      {
        "clash_mode": "Direct",
        "outbound": "direct"
      },
      {
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
        "rule_set": "geoip-ru",
        "outbound": "direct"
      },
      {
        "clash_mode": "Global",
        "outbound": "proxy"
      }
    ],
    "rule_set": [
      {
        "tag": "adguard",
        "type": "remote",
        "url": "https://github.com/jinndi/adguard-filter-list-srs/releases/latest/download/adguard-filter-list.srs",
        "download_detour": "direct"
      },
      {
        "tag": "fakeip-filter",
        "type": "remote",
        "url": "https://github.com/jinndi/fakeip-filter-srs/releases/latest/download/fakeip-filter.srs",
        "download_detour": "direct"
      },
      {
        "tag": "category-ip-geo-detect",
        "type": "remote",
        "url": "https://github.com/KaringX/karing-ruleset/raw/sing/geo/geosite/category-ip-geo-detect.srs",
        "download_detour": "direct"
      },
      {
        "tag": "geosite-cheburnet",
        "type": "remote",
        "url": "https://github.com/jinndi/geosite-cheburnet/releases/latest/download/geosite-cheburnet.srs",
        "download_detour": "direct"
      },
      {
        "tag": "geoip-ru",
        "type": "remote",
        "url": "https://github.com/KaringX/karing-ruleset/raw/sing/geo/geoip/ru.srs",
        "download_detour": "direct"
      },
      {
        "tag": "geosite-category-ru",
        "type": "remote",
        "url": "https://github.com/KaringX/karing-ruleset/raw/sing/geo/geosite/category-ru.srs",
        "download_detour": "direct"
      }
    ]
  },
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

> Если возникают проблемы со звонками, удалите из секции `route.rules` следующий блок (действие выполняется на ваш страх и риск):

```json
  {
    "protocol": "stun",
    "action": "reject"
  },
```

  - **`File` `Script Operator`**: Там же, но чуть ниже будет карточка с заголовком `Add an action`. В ней выбираем `Script Operator`. Появится новое окно ввода выше - переключаемся в нём на вкладку `Local Content` и вставляем следующий шаблон:

```javascript

//// Указываем имя вашей созданной ранее подписки ('mysub')
const subName = "mysub"

// 1. Загружаем прокси из вашей подписки
let singboxProxies = []
try {
  singboxProxies = await produceArtifact({
    type: "subscription", // доступно так-же 'collection'
    name: subName,
    platform: "sing-box",
    produceType: "internal"
  })
} catch (e) {
  throw new Error(`Не удалось загрузить подписку '${subName}'. Проверьте имя вкладки подписки.`)
}

// 2. Парсим шаблон (вставленный ранее как основа конфига)
let config
try {
  config = JSON.parse($files[0])
} catch (e) {
  throw new Error("Ошибка парсинга шаблона: " + e.message)
}

// Инициализируем массив outbounds, если он не существует
if (!config.outbounds) {
  config.outbounds = []
}

// Извлекаем только имена (теги) всех прокси, чтобы добавить их в группы
let allProxyTags = singboxProxies.map(p => p.tag)

if (allProxyTags.length > 0) {
  // Создаем группу 1: URL-Test (Автовыбор сервера с наименьшим пингом)
  let urlTestGroup = {
    "type": "urltest",
    "tag": "⚡ Auto Select",
    "outbounds": [...allProxyTags],
    "url": "https://www.gstatic.com/generate_204",
    "interrupt_exist_connections": true,
    "interval": "10m", // тут можно указать как часто делать проверку
    "tolerance": 150   // если на это кол-во пинг найдется ниже,
                       // то будет переключение на этот сервер
  };

  // Создаем группу 2: Главный селектор "proxy" (основной тег для прокси)
  // В него добавляем сначала direct, затем автовыбор, а потом список всех серверов поштучно
  let mainSelector = {
    "type": "selector",
    "tag": "proxy",
    "default": "⚡ Auto Select",
    "interrupt_exist_connections": true,
    "outbounds": [
      "⚡ Auto Select",
      ...allProxyTags
    ]
  }

  // Добавляем созданные группы в начало outbounds
  config.outbounds.push(mainSelector)
  config.outbounds.push(urlTestGroup)
} else {
  // Если подписка пуста, создаем заглушку-селектор proxy, чтобы конфиг не ломался
  config.outbounds.push({
    "type": "selector",
    "tag": "proxy",
    "outbounds": ["direct"]
  })
}

// 3. Добавляем в самый конец реальные прокси-серверы из подписки
config.outbounds.push(...singboxProxies)

// 4. Добавляем обязательный direct outbound, если его нет
if (!config.outbounds.some(o => o.tag === 'direct')) {
  config.outbounds.push({ "type": "direct", "tag": "direct" })
}

// Результат отдаем дальше
$content = JSON.stringify(config, null, 2)
```

В этом шаблоне требуется только укзатаь имя подписки в:

```javascript
//// Указываем имя вашей созданной ранее подписки ('mysub')
const subName = "mysub"
```

После чего сохраните ваш файл кнопкой `Save` внизу.

  - **`Share`**: Последний этап - создание ссылки на готовую подписку (`File`). Переходим во вкладку `Share`, нажимаем на кнопку `Create Now`. Далее в появившемся окне в поле `Source` выбираем `File`, а затем - ваш созданный в предыдущем пункте файл (`myfile`). Задаём срок действия в поле `Valid for` (количество дней/месяцев и т. д. в зависимости от выбранного режима в `Expiration Mode`) и нажимаем `Save Changes`. Подписка готова! Она появится в списке вкладки `Share`, скопировать ссылку можно нажатием на значок копирования.


3. Скачиваем из репозитория [sing-box](https://github.com/SagerNet/sing-box/releases/latest) последнюю стабильную APK-версию приложения для Android (пример:  
   `SFA-1.13.12-arm64-v8a.apk`) и устанавливаем!


4. Приложения, которым вы не доверяете, могут собирать сведения об установленных на вашем устройстве приложениях. Поэтому мы скрываем доверенные приложения, включая установленное приложение sing-box, с помощью настроек телефона. Например, на Realme это находится в разделе:

**Настройки → Безопасность и конфиденциальность → Скрыть приложения**

Все остальные VPN-приложения удаляем.

5. В приложении sing-box добавляем вашу ссылку на подписку, подключаемся и радуемся, но не сильно.

~~Бит лютый вообще...~~