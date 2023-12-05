---
layout: default
title: 定制品牌
nav_order: 130
---

# 定制品牌
引入了1.2
{: .label .label-purple }

默认情况下，OpenSearch仪表板使用OpenSearch徽标，但是如果要使用自定义的品牌元素，例如Favicon或Main Dashboards徽标，则可以通过编辑来做到这一点`opensearch_dashboards.yml` 或包括自定义`opensearch_dashboards.yml` 启动OpenSearch cluster时文件。

例如，如果您使用Docker启动OpenSearch群集，请在`opensearch-dashboards` 您的部分`docker-compose.yml` 文件：

```
volumes:
  - ./opensearch_dashboards.yml:/usr/share/opensearch-dashboards/config/opensearch_dashboards.yml
```

这样做可以替换Docker映像的默认值`opensearch_dashboards.yml` 习惯`opensearch_dashboards.yml` 文件，因此请确保还包括您所需的设置。例如，如果要为OpenSearch仪表板配置TLS，请参见[为OpenSearch仪表板配置TLS]({{site.url}}{{site.baseurl}}/dashboards/install/tls)。

关于-启动OpenSearch仪表板，OpenSearch仪表板现在使用您的自定义元素。

## 品牌元素

OpenSearch仪表板中的以下元素可自定义：

![OpenSearch可自定义的品牌元素]({{site.url}}{{site.baseurl}}/images/dashboards-branding-labels.png)

环境| 相应的品牌元素
:--- | :---
标识| 标题徽标。看#图像中的1。
标记| OpenSearch仪表板标记。看#图像中的2。
LoadingLogo| OpenSearch仪表板启动时使用的加载徽标。看#图像中的3。
Faviconurl| 网站图标。加载应用程序标题旁边。看#图像中的4。
ApplicationTitle| 应用程序的标题。看#图像中的5。

要合并导航控件并减少标头在页面上占用的空间，请参见[凝结的标头](#condensed-header)。
{: .note}

要开始在OpenSearch仪表板中使用您自己的品牌元素`opensearch_dashboards.yml`：

```yml
# opensearchDashboards.branding:
  # logo:
    # defaultUrl: ""
    # darkModeUrl: ""
  # mark:
    # defaultUrl: ""
    # darkModeUrl: ""
  # loadingLogo:
    # defaultUrl: ""
    # darkModeUrl: ""
  # faviconUrl: ""
  # applicationTitle: ""
```

将要用作品牌元素的URL添加到适当的设置中。有效的图像类型是`SVG`，，，，`PNG`， 和`GIF`。

还提供了深色模式仪表板的自定义，但是您首先必须提供有效的链接`defaultUrl`，然后链接到您首选的图像与`darkModeUrl`。如果您不提供`darkModeUrl` 链接，然后仪表板使用提供的`defaultUrl` 黑暗模式的元素。您无需自定义所有品牌元素，因此，如果您愿意，只需更改徽标或任何其他元素是完全有效的。如评论，留下不变的元素。

以下示例演示了如何使用`SVG` 文件作为徽标，但将其他元素作为默认值留下。

```yml
logo:
  defaultUrl: "https://example.com/validUrl.svg"
  darkModeUrl: "https://example.com/validDarkModeUrl.svg"
# mark:
#   defaultUrl: ""
#   darkModeUrl: ""
# loadingLogo:
#   defaultUrl: ""
#   darkModeUrl: ""
# faviconUrl: ""
applicationTitle: "My custom application"
```

我们建议链接到Web服务器上托管的图像，但是如果您确实想使用本地托管的图像，请将图像保存在其中`assets`，然后配置`opensearch_dashboards.yml` 使用正确的路径。您可以通过`ui/assets` 文件夹。

下面的示例假定仪表板使用并演示如何链接到本地存储的图像的5601的默认端口。

```yml
logo:
  defaultUrl: "https://localhost:5601/ui/assets/my-own-image.svg"
  darkModeUrl: "https://localhost:5601/ui/assets/dark-mode-my-own-image.svg"
mark:
  defaultUrl: "https://localhost:5601/ui/assets/my-own-image2.svg"
  darkModeUrl: "https://localhost:5601/ui/assets/dark-mode-my-own-image2.svg"
# loadingLogo:
#   defaultUrl: ""
#   darkModeUrl: ""
# faviconUrl: ""
applicationTitle: "My custom application"
```

### 凝结的标头

凝结的标头视图通过将导航元素组合到单个标头栏中，减少了标头的足迹，并释放了页面上的空间。

当前的默认视图在外观上保持接近-Bar标头在先前版本的仪表板上提供，差异很小。要指定冷凝标头，请添加配置属性`useExpandedHeader` 到`opensearch_dashboards.yml` 提交并将值设置为`false`，如以下示例所示。

 ```YML
# OpenSearchDashboards.Branding：
  # 标识：
    Defaulturl："https://example.com/sample.svg"
    Darkmodeurl："https://example.com/dark-mode-sample.svg"
  # 标记：
    # Defaulturl：""
    # Darkmodeurl：""
  # LoadingLogo：
    # Defaulturl：""
    # Darkmodeurl：""
  # Faviconurl：""
  ApplicationTitle："my custom application"
  USE ExpandedHeader：false
```

In a future release, default behavior will become `useExpandedHeader: false`. If you want to retain the default view in subsequent releases, you can explicitly set the property to `true` in advance. Alternatively, you can also do this when upgrading.
{: .note }

The condensed view header appears as in the example below.

![Condensed header]({{site.url}}{{site.baseurl}}/images/DBs-Condensed.jpeg)

Header element | Description
:--- | :---
OpenSearch logo | See #1. Functions as the home button.
Header bar | See #2. A single header bar used for all navigation controls.

The default view remains close to the traditional view, with minor changes.

![Default header]({{site.url}}{{site.baseurl}}/images/DBs-Traditional.jpeg)

Header element | Description
:--- | :---
Home button | See #1. Returns to the home page and provides an indication when a page is loading.
Header label | See #2. The label also functions as a home button.
Navigation controls | See #3. Additional navigation controls on right-side insertion points.

#### Preserving nagivation elements in the default view

You can continue using the top header bar in the default view for custom navigation links (such as menu items and plugins). Follow the steps below to keep these elements in the top header in the default view.
1. Replace the property `coreStart.chrome.navControls.registerRight(...)` with `coreStart.chrome.navControls.registerExpandedRight(...)` and then replace the property  `coreStart.chrome.navControls.registerCenter(...)` with `coreStart.chrome.navControls.registerExpandedCenter(...)`

2. Make sure the configuration property `useExpandedHeader` is explicitly set to `true`.


## Sample configuration

The following configuration enables the Security plugin and SSL within OpenSearch Dashboards and uses custom branding elements to replace the OpenSearch logo and application title.

```yml
server.host："0"
OpenSearch.Hosts：["https://localhost:9200"这是给出的
opensearch.ssl.verificationmode：无
opensearch.username："kibanaserver"
OpenSearch.Password："kibanaserver"
opensearch.requestheaderslowerlist：[授权，安全士兵]
#server.ssl.enabled: 真的
#server.ssl.certificate: /path/to/your/server/证书
#server.ssl.key: /path/to/your/server/键

opensearch_security.multitenancy.enabled：true
opensearch_security.multitenancy.tenants.preferred：["Private"，，，，"Global"这是给出的
opensearch_security.readonly_mode.roles：[["kibana_read_only"这是给出的
# 如果您正在运行OpenSearch，请使用此设置-仪表板没有HTTPS
opensearch_security.cookie.secure：false

OpenSearchDashboards.Branding：
  标识：
    Defaulturl："https://example.com/sample.svg"
    Darkmodeurl："https://example.com/dark-mode-sample.svg"
  # 标记：
  #   Defaulturl：""
  #   Darkmodeurl：""
  # LoadingLogo：
  #   Defaulturl：""
  #   Darkmodeurl：""
  # Faviconurl：""
  ApplicationTitle："Just some testing"
```

