## **wordpress 执行顺序**

### **1.STEP 1：加载 index.php 文件**
----

```php
<?php

#定义是否加载主题文件，true为加载；
define( 'WP_USE_THEMES', true );

#加载wp-blog-header.php文件，该文件用于启动WordPress环境及模板；
require __DIR__ . '/wp-blog-header.php';
```

### **STEP 2：加载 wp-blog-header.php 文件**
----

```php

<?php

# 判断$wp_did_header变量是否已经设置，如未设置则执行代码块；
if ( ! isset( $wp_did_header ) ) {
    # 见解析1；
	$wp_did_header = true;

	# 见解析2；
	require_once __DIR__ . '/wp-load.php';

	# 见解析3；
	wp();

	# 见解析4；
	require_once ABSPATH . WPINC . '/template-loader.php';

}

```
>解析1： 对 `$wp_did_header` 进行赋值，如果代码块已经执行过，判断就会失败，代码块就不会再执行。这种做法可以确保 `wp-blog-header`.php 文件只执行一次（重复执行的话会出现函数名冲突、变量重置等，WordPress 会崩溃的！）；

>解析2： 加载 WordPress 根目录下 `wp-load.php` 文件，执行初始化工作，如初始化常量、环境、加载类库和核心代码等完成 WordPress 环境启动工作，如加载 `wp-includes` 目录下 `functions.php`（函数库）、`class-wp.php`（类库）、`plugin.php`（插件）、`pomo`目录（语言包）、`query.php`（数据请求）、`theme.php`（加载主题文件）、`post-template.php`（文章模板）、`comment.php`（评论模板）、`rewrite.php`（URL重写）等等；


>解析3： 执行 wp() 函数，执行内容处理工作，如根据用户的请求调用相关函数获取和处理数据，为前端展示准备数据；

>解析4： 加载根目录绝对路径下 `wp-includes` 目录中 template-loader.php 文件，执行主题应用工作，如根据用户的请求加载主题模板。


* wp-load.php 会完成页面生成所需要的所有环境、变量、API等，相当于做了好准备工作；
* wp() 函数根据用户请求的URL从数据库中取出相应的数据内容备用；
* template-loader.php 把已经准备好的内容用主题所设定的样式展现方式给拼接出来。这三项工作完成，就可以将用户请求的页面展现出来。

---

### **STEP 3：加载 wp-load.php 文件（初始化）**
```php
<?php

#定义常量 ABSPATH 为根目录绝对地址
if ( ! defined( 'ABSPATH' ) ) {
	define( 'ABSPATH', __DIR__ . '/' );
}

#加载根目录下 wp-config.php 文件；
require_once dirname( ABSPATH ) . '/wp-config.php';

```

#### 3.1 加载 wp-config.php 文件

#### 3.2 加载 wp-settings.php 文件
相关源码如下：
```php
<?php
/**
 *主要用于创建和定义常见变量、函数和类的库来为WordPress运行做准备，
 *也就是说WordPress运行过程中使用的大多数变量、函数和类等核心代码
 *都是在这*个文件中定义的。这个文件相当于一个总控制器，很多常量定
 *义、函数定义等都是在其他文件中完成，而该文件的作用就是执行那些文
 *件或执行在那些文件中已经定义好的函数
 */

/**
 * 定义常量 WPINC
 * 存储函数、类和核心内容的WordPress目录的位置
 */
define( 'WPINC', 'wp-includes' );

/**
 * 当前WordPress版本信息.
 *
 *
 * @global string $wp_version             WordPress 版本.
 * @global int    $wp_db_version          WordPress 数据库版本.
 * @global string $tinymce_version        TinyMCE 版本.
 * @global string $required_php_version   所需的PHP版本.
 * @global string $required_mysql_version 所需的MYSQL版本.
 * @global string $wp_local_package       区域设置.
 */
global $wp_version, $wp_db_version, $tinymce_version, $required_php_version, $required_mysql_version, $wp_local_package;
require ABSPATH . WPINC . '/version.php';
require ABSPATH . WPINC . '/load.php';

// 检查所需的PHP版本、MySQL扩展或数据库插件.
wp_check_php_mysql_versions();

// 加载初始化所需的文件.
require ABSPATH . WPINC . '/class-wp-paused-extensions-storage.php';
require ABSPATH . WPINC . '/class-wp-fatal-error-handler.php';
require ABSPATH . WPINC . '/class-wp-recovery-mode-cookie-service.php';
require ABSPATH . WPINC . '/class-wp-recovery-mode-key-service.php';
require ABSPATH . WPINC . '/class-wp-recovery-mode-link-service.php';
require ABSPATH . WPINC . '/class-wp-recovery-mode-email-service.php';
require ABSPATH . WPINC . '/class-wp-recovery-mode.php';
require ABSPATH . WPINC . '/error-protection.php';
require ABSPATH . WPINC . '/default-constants.php';
require_once ABSPATH . WPINC . '/plugin.php';

/**
 * 如果尚未配置, `$blog_id` 在单页站点设置为默认值1
 * 在多站点中, 通过配置 ms-settings.php `$blog_id` 的值将会被覆盖.
 */
global $blog_id;

// 设置初始默认常量，包括WP_MEMORY_LIMIT、WP_MAX_MEMORY_LIMIT、WP_DEBUG、SCRIPT_DEBUG、WP_CONTENT_DIR和WP_CACHE.
wp_initial_constants();

// 确保我们尽快为致命错误注册关闭处理程序.
wp_register_fatal_error_handler();

// WordPress 从 UTC 计算偏移量.
date_default_timezone_set( 'UTC' );

// 关闭注册为全局变量.
wp_unregister_GLOBALS();

// 规范 $_SERVER 变量设置.
wp_fix_server_vars();

// 检查我们是否处于维护模式.
wp_maintenance();

// 开始加载计时器.
timer_start();

// 检查我们是否处于调试模式.
wp_debug_mode();

/**
 *
 * 此过滤器在插件使用之前运行。它是为非 web 运行时设计的。
 * 如果返回 false，则永远不会加载 advanced-cache.php
 *
 */
if ( WP_CACHE && apply_filters( 'enable_loading_advanced_cache_dropin', true ) && file_exists( WP_CONTENT_DIR . '/advanced-cache.php' ) ) {
	// 用于高级缓存插件。使用静态插入，因为您只需要一个.
	include WP_CONTENT_DIR . '/advanced-cache.php';

	// 重新初始化由 advanced-cache.php 手动添加的任何挂钩.
	if ( $wp_filter ) {
		$wp_filter = WP_Hook::build_preinitialized_hooks( $wp_filter );
	}
}

// 如果未设置，定义 WP_LANG_DIR.
wp_set_lang_dir();

// 加载前期 WordPress 文件.
require ABSPATH . WPINC . '/compat.php'; // 提供某些 PHP 版本缺少的函数（用于支持不同版本 PHP 上的兼容和移植）
require ABSPATH . WPINC . '/class-wp-list-util.php';
require ABSPATH . WPINC . '/formatting.php';
require ABSPATH . WPINC . '/meta.php';
require ABSPATH . WPINC . '/functions.php'; //定义 WordPress 主要的函数 API
require ABSPATH . WPINC . '/class-wp-meta-query.php';
require ABSPATH . WPINC . '/class-wp-matchesmapregex.php';
require ABSPATH . WPINC . '/class-wp.php';
require ABSPATH . WPINC . '/class-wp-error.php';
require ABSPATH . WPINC . '/pomo/mo.php';

/**
 * @global wpdb $wpdb WordPress 数据库抽象对象.
 */
global $wpdb;
// 加载 wpdb 类，和顺道加载 db.php (如果存在)
require_wp_db();

// 为数据库表列设置数据库表前缀和格式说明符.
$GLOBALS['table_prefix'] = $table_prefix;
wp_set_wpdb_vars();

// 开启 WordPress 对象缓存，或者扩展对象缓存（如果相应 drop-in 存在的话）.
wp_start_object_cache();

// 附加默认过滤器.
require ABSPATH . WPINC . '/default-filters.php';

// 初始化多站点（如果开启多站点）.
if ( is_multisite() ) {
	require ABSPATH . WPINC . '/class-wp-site-query.php';
	require ABSPATH . WPINC . '/class-wp-network-query.php';
	require ABSPATH . WPINC . '/ms-blogs.php';
	require ABSPATH . WPINC . '/ms-settings.php';
} elseif ( ! defined( 'MULTISITE' ) ) {
	define( 'MULTISITE', false );
}

register_shutdown_function( 'shutdown_action_hook' );

// 如果我们只需要基本的东西，就不要再加载 WordPress.
if ( SHORTINIT ) {
	return false;
}

// 加载L10n库.
require_once ABSPATH . WPINC . '/l10n.php';
require_once ABSPATH . WPINC . '/class-wp-locale.php';
require_once ABSPATH . WPINC . '/class-wp-locale-switcher.php';

// 如果没有安装 WordPress，运行安装程序.
wp_not_installed();

// 加载大部分 WordPress 周边.
require ABSPATH . WPINC . '/class-wp-walker.php';
require ABSPATH . WPINC . '/class-wp-ajax-response.php';
require ABSPATH . WPINC . '/capabilities.php';
require ABSPATH . WPINC . '/class-wp-roles.php';
require ABSPATH . WPINC . '/class-wp-role.php';
require ABSPATH . WPINC . '/class-wp-user.php';
require ABSPATH . WPINC . '/class-wp-query.php';
require ABSPATH . WPINC . '/query.php';
require ABSPATH . WPINC . '/class-wp-date-query.php';
require ABSPATH . WPINC . '/theme.php';
require ABSPATH . WPINC . '/class-wp-theme.php';
require ABSPATH . WPINC . '/template.php';
require ABSPATH . WPINC . '/class-wp-user-request.php';
require ABSPATH . WPINC . '/user.php';
require ABSPATH . WPINC . '/class-wp-user-query.php';
require ABSPATH . WPINC . '/class-wp-session-tokens.php';
require ABSPATH . WPINC . '/class-wp-user-meta-session-tokens.php';
require ABSPATH . WPINC . '/class-wp-metadata-lazyloader.php';
require ABSPATH . WPINC . '/general-template.php';
require ABSPATH . WPINC . '/link-template.php';
require ABSPATH . WPINC . '/author-template.php';
require ABSPATH . WPINC . '/post.php';
require ABSPATH . WPINC . '/class-walker-page.php';
require ABSPATH . WPINC . '/class-walker-page-dropdown.php';
require ABSPATH . WPINC . '/class-wp-post-type.php';
require ABSPATH . WPINC . '/class-wp-post.php';
require ABSPATH . WPINC . '/post-template.php';
require ABSPATH . WPINC . '/revision.php';
require ABSPATH . WPINC . '/post-formats.php';
require ABSPATH . WPINC . '/post-thumbnail-template.php';
require ABSPATH . WPINC . '/category.php';
require ABSPATH . WPINC . '/class-walker-category.php';
require ABSPATH . WPINC . '/class-walker-category-dropdown.php';
require ABSPATH . WPINC . '/category-template.php';
require ABSPATH . WPINC . '/comment.php';
require ABSPATH . WPINC . '/class-wp-comment.php';
require ABSPATH . WPINC . '/class-wp-comment-query.php';
require ABSPATH . WPINC . '/class-walker-comment.php';
require ABSPATH . WPINC . '/comment-template.php';
require ABSPATH . WPINC . '/rewrite.php';
require ABSPATH . WPINC . '/class-wp-rewrite.php';
require ABSPATH . WPINC . '/feed.php';
require ABSPATH . WPINC . '/bookmark.php';
require ABSPATH . WPINC . '/bookmark-template.php';
require ABSPATH . WPINC . '/kses.php';
require ABSPATH . WPINC . '/cron.php';
require ABSPATH . WPINC . '/deprecated.php';
require ABSPATH . WPINC . '/script-loader.php';
require ABSPATH . WPINC . '/taxonomy.php';
require ABSPATH . WPINC . '/class-wp-taxonomy.php';
require ABSPATH . WPINC . '/class-wp-term.php';
require ABSPATH . WPINC . '/class-wp-term-query.php';
require ABSPATH . WPINC . '/class-wp-tax-query.php';
require ABSPATH . WPINC . '/update.php';
require ABSPATH . WPINC . '/canonical.php';
require ABSPATH . WPINC . '/shortcodes.php';
require ABSPATH . WPINC . '/embed.php';
require ABSPATH . WPINC . '/class-wp-embed.php';
require ABSPATH . WPINC . '/class-wp-oembed.php';
require ABSPATH . WPINC . '/class-wp-oembed-controller.php';
require ABSPATH . WPINC . '/media.php';
require ABSPATH . WPINC . '/http.php';
require ABSPATH . WPINC . '/class-http.php';
require ABSPATH . WPINC . '/class-wp-http-streams.php';
require ABSPATH . WPINC . '/class-wp-http-curl.php';
require ABSPATH . WPINC . '/class-wp-http-proxy.php';
require ABSPATH . WPINC . '/class-wp-http-cookie.php';
require ABSPATH . WPINC . '/class-wp-http-encoding.php';
require ABSPATH . WPINC . '/class-wp-http-response.php';
require ABSPATH . WPINC . '/class-wp-http-requests-response.php';
require ABSPATH . WPINC . '/class-wp-http-requests-hooks.php';
require ABSPATH . WPINC . '/widgets.php';
require ABSPATH . WPINC . '/class-wp-widget.php';
require ABSPATH . WPINC . '/class-wp-widget-factory.php';
require ABSPATH . WPINC . '/nav-menu.php';
require ABSPATH . WPINC . '/nav-menu-template.php';
require ABSPATH . WPINC . '/admin-bar.php';
require ABSPATH . WPINC . '/rest-api.php';
require ABSPATH . WPINC . '/rest-api/class-wp-rest-server.php';
require ABSPATH . WPINC . '/rest-api/class-wp-rest-response.php';
require ABSPATH . WPINC . '/rest-api/class-wp-rest-request.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-controller.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-posts-controller.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-attachments-controller.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-post-types-controller.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-post-statuses-controller.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-revisions-controller.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-autosaves-controller.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-taxonomies-controller.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-terms-controller.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-users-controller.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-comments-controller.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-search-controller.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-blocks-controller.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-block-renderer-controller.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-settings-controller.php';
require ABSPATH . WPINC . '/rest-api/endpoints/class-wp-rest-themes-controller.php';
require ABSPATH . WPINC . '/rest-api/fields/class-wp-rest-meta-fields.php';
require ABSPATH . WPINC . '/rest-api/fields/class-wp-rest-comment-meta-fields.php';
require ABSPATH . WPINC . '/rest-api/fields/class-wp-rest-post-meta-fields.php';
require ABSPATH . WPINC . '/rest-api/fields/class-wp-rest-term-meta-fields.php';
require ABSPATH . WPINC . '/rest-api/fields/class-wp-rest-user-meta-fields.php';
require ABSPATH . WPINC . '/rest-api/search/class-wp-rest-search-handler.php';
require ABSPATH . WPINC . '/rest-api/search/class-wp-rest-post-search-handler.php';
require ABSPATH . WPINC . '/class-wp-block-type.php';
require ABSPATH . WPINC . '/class-wp-block-styles-registry.php';
require ABSPATH . WPINC . '/class-wp-block-type-registry.php';
require ABSPATH . WPINC . '/class-wp-block-parser.php';
require ABSPATH . WPINC . '/blocks.php';
require ABSPATH . WPINC . '/blocks/archives.php';
require ABSPATH . WPINC . '/blocks/block.php';
require ABSPATH . WPINC . '/blocks/calendar.php';
require ABSPATH . WPINC . '/blocks/categories.php';
require ABSPATH . WPINC . '/blocks/latest-comments.php';
require ABSPATH . WPINC . '/blocks/latest-posts.php';
require ABSPATH . WPINC . '/blocks/rss.php';
require ABSPATH . WPINC . '/blocks/search.php';
require ABSPATH . WPINC . '/blocks/shortcode.php';
require ABSPATH . WPINC . '/blocks/social-link.php';
require ABSPATH . WPINC . '/blocks/tag-cloud.php';

$GLOBALS['wp_embed'] = new WP_Embed();

// 加载多站点特定文件.
if ( is_multisite() ) {
	require ABSPATH . WPINC . '/ms-functions.php';
	require ABSPATH . WPINC . '/ms-default-filters.php';
	require ABSPATH . WPINC . '/ms-deprecated.php';
}

// 定义依赖 API 获取默认值的常量.
// 定义必须使用 plugin 目录常量，该常量可能在 sunrise.php 中被重写.
wp_plugin_directory_constants();

$GLOBALS['wp_plugin_paths'] = array();

// 加载必须使用插件.
foreach ( wp_get_mu_plugins() as $mu_plugin ) {;
	include_once $mu_plugin;
	do_action( 'mu_plugin_loaded', $mu_plugin );
}
unset( $mu_plugin );

// 加载网络激活插件.
if ( is_multisite() ) {
	foreach ( wp_get_active_network_plugins() as $network_plugin ) {
		wp_register_plugin_realpath( $network_plugin );
		include_once $network_plugin;
		do_action( 'network_plugin_loaded', $network_plugin );
	}
	unset( $network_plugin );
}

/**
 * 必须使用且网络激活的插件已加载后立即激发.
 */
do_action( 'muplugins_loaded' );

if ( is_multisite() ) {
	ms_cookie_constants();
}

// 加载多站点后定义常量.
wp_cookie_constants();

// 定义并实施SSL常量.
wp_ssl_constants();

// 创建通用全局变量.
require ABSPATH . WPINC . '/vars.php';

// 使分类和帖子可用于插件和主题.
// @plugin authors: warning: 它们在init钩子上再次注册.
create_initial_taxonomies();
create_initial_post_types();

wp_start_scraping_edited_file_errors();

// 注册默认主题目录根目录.
register_theme_directory( get_theme_root() );

if ( ! is_multisite() ) {
	// 处理请求恢复模式链接并启动恢复模式的用户.
	wp_recovery_mode()->initialize();
}

// 创建 WP_Site_Health 的实例，以便 Cron 事件可以触发
if ( ! class_exists( 'WP_Site_Health' ) ) {
	require_once ABSPATH . 'wp-admin/includes/class-wp-site-health.php';
}
WP_Site_Health::get_instance();

// 加载已启用插件.
foreach ( wp_get_active_and_valid_plugins() as $plugin ) {
	wp_register_plugin_realpath( $plugin );
	include_once $plugin;
	do_action( 'plugin_loaded', $plugin );
}
unset( $plugin );

// 加载可插拔功能.
require ABSPATH . WPINC . '/pluggable.php';
require ABSPATH . WPINC . '/pluggable-deprecated.php';

// 设置内部编码.
wp_set_internal_encoding();

// 如果对象缓存已启用且函数存在，则运行wp_cache_postload()
if ( WP_CACHE && function_exists( 'wp_cache_postload' ) ) {
	wp_cache_postload();
}

do_action( 'plugins_loaded' );

// 定义影响功能的常量（如果尚未定义）.
wp_functionality_constants();

// 添加魔法引号并设置 $_REQUEST ( $_GET + $_POST ).
wp_magic_quotes();

/**
 * 在清理注释 cookie 时激发.
 */
do_action( 'sanitize_comment_cookies' );

/**
 * WordPress 查询对象
 */
$GLOBALS['wp_the_query'] = new WP_Query();

/**
 * 使用此全局设置进行WordPress查询
 */
$GLOBALS['wp_query'] = $GLOBALS['wp_the_query'];

/**
 * 保存 WordPress Rewrite 对象以创建漂亮的 url
 */
$GLOBALS['wp_rewrite'] = new WP_Rewrite();

/**
 * WordPress 对象
 */
$GLOBALS['wp'] = new WP();

/**
 * WordPress 小部件工厂对象
 */
$GLOBALS['wp_widget_factory'] = new WP_Widget_Factory();

/**
 * WordPress 用户角色
 */
$GLOBALS['wp_roles'] = new WP_Roles();

/**
 * 在加载主题之前触发.
 */
do_action( 'setup_theme' );

// 定义与模板相关的常量.
wp_templating_constants();

// 加载默认文本 本地化域.
load_default_textdomain();

$locale      = get_locale();
$locale_file = WP_LANG_DIR . "/$locale.php";
if ( ( 0 === validate_file( $locale ) ) && is_readable( $locale_file ) ) {
	require $locale_file;
}
unset( $locale_file );

/**
 * 加载区域设置域日期和各种字符串的 WordPress 区域设置对象.
 */
$GLOBALS['wp_locale'] = new WP_Locale();

/**
 * 用于切换区域设置的 WordPress 区域设置切换器对象.
 */
$GLOBALS['wp_locale_switcher'] = new WP_Locale_Switcher();
$GLOBALS['wp_locale_switcher']->init();

//加载已启用主题的函数，父主题和子主题（如果适用）.
foreach ( wp_get_active_and_valid_themes() as $theme ) {
	if ( file_exists( $theme . '/functions.php' ) ) {
		include $theme . '/functions.php';
	}
}
unset( $theme );

/**
 * 加载主题后触发.
 */
do_action( 'after_setup_theme' );

// 设置当前用户.
$GLOBALS['wp']->init();

/**
 * 在WordPress完成加载但在发送任何头文件之前触发
 *
 * 在这个阶段 WP 的大部分已被加载，用户也已认证。
 * WP 会继续加载 init 钩子上的挂载者。
 *
 * 如果你想在WP加载后插入一个action，请使用下面的 wp_loaded 钩子。
 */
do_action( 'init' );

// 检查站点状态.
if ( is_multisite() ) {
	$file = ms_site_check();
	if ( true !== $file ) {
		require $file;
		die();
	}
	unset( $file );
}

/**
 * 当WP、所有插件以及主题都被完全加载和实例化后，该钩子将被解除。
 * Ajax请求应该使用 wp-admin/admin-ajax.php，admin-ajax.php 
 * 能够处理未登录用户的请求
 */
do_action( 'wp_loaded' );
```


----

### **STEP 4： 执行 wp() 函数（内容处理，位于 wp-includes 下的 functions.php）**

```php
function wp( $query_vars = '' ) {

    # 见解析1
    global $wp, $wp_query, $wp_the_query;
    
    # 见解析2
    $wp->main( $query_vars );
    
    # 见解析3
	if ( ! isset( $wp_the_query ) ) {
		$wp_the_query = $wp_query;
	}
}
```

>解析1： 对变量 `$wp`，`$wp_query`，`$wp_the_query` 进行全局化；

>解析2： 调用 `$wp->main()`，即调用对象 $wp 的 main() 方法，该对象是 `class-wp.php` 文件中 WP 类实例化得到的，该类主要用于启动 `WordPress` 环境；

>解析3： 判断 `$wp_the_query` 是否设置，若未设置将其赋值为 `$wp_query`，该对象是 `query.php` 文件中 `WP_Query` 类实例化得到的，该类作用强大，几乎 WP 所需要的所有数据信息都是由该类得到的，所以内容的准备工作基本都是这段代码来完成的；

**至此，WP 根据请求准备相应数据的工作也已经完成，下面就需要加载模板并把这些数据展现到前台去了。**


### **STEP 5： 加载 template-loader.php 文件（主题应用）**

```php
<?php
/**
 * 根据访问者的 url 加载正确的模板
 */
if ( wp_using_themes() ) {
	/**
	 * 在确定要加载哪个模板之前触发
	 */
	do_action( 'template_redirect' );
}

/**
 * 是否允许 HEAD 请求生成内容 （过滤器）
 */
if ( 'HEAD' === $_SERVER['REQUEST_METHOD'] && apply_filters( 'exit_on_http_head', true ) ) {
	exit();
}

// 即使不使用主题，也要处理 feeds 和 trackback.
if ( is_robots() ) {
	/**
	 * 当模板加载程序确定 robots.txt 请求时触发.
	 */
	do_action( 'do_robots' );
	return;
} elseif ( is_favicon() ) {
	/**
	 * 当模板加载程序确定 favicon.ico 请求时触发.
	 */
	do_action( 'do_favicon' );
	return;
} elseif ( is_feed() ) {
	do_feed();
	return;
} elseif ( is_trackback() ) {
	require ABSPATH . 'wp-trackback.php';
	return;
}
# 见解析1
if ( wp_using_themes() ) {

	$tag_templates = array(
		'is_embed'             => 'get_embed_template',
		'is_404'               => 'get_404_template',
		'is_search'            => 'get_search_template',
		'is_front_page'        => 'get_front_page_template',
		'is_home'              => 'get_home_template',
		'is_privacy_policy'    => 'get_privacy_policy_template',
		'is_post_type_archive' => 'get_post_type_archive_template',
		'is_tax'               => 'get_taxonomy_template',
		'is_attachment'        => 'get_attachment_template',
		'is_single'            => 'get_single_template',
		'is_page'              => 'get_page_template',
		'is_singular'          => 'get_singular_template',
		'is_category'          => 'get_category_template',
		'is_tag'               => 'get_tag_template',
		'is_author'            => 'get_author_template',
		'is_date'              => 'get_date_template',
		'is_archive'           => 'get_archive_template',
	);
	$template      = false;

	// 遍历每个模板条件，并找到适当的模板文件.
	foreach ( $tag_templates as $tag => $template_getter ) {
		if ( call_user_func( $tag ) ) {
			$template = call_user_func( $template_getter );
		}

		if ( $template ) {
			if ( 'is_attachment' === $tag ) {
				remove_filter( 'the_content', 'prepend_attachment' );
			}

			break;
		}
	}

	if ( ! $template ) {
		$template = get_index_template();
	}

	/**
	 * 在引入当前模板之前筛选其路径
	 */
	$template = apply_filters( 'template_include', $template );
	if ( $template ) {
		include $template;
	} elseif ( current_user_can( 'switch_themes' ) ) {
		$theme = wp_get_theme();
		if ( $theme->errors() ) {
			wp_die( $theme->errors() );
		}
	}
	return;
}
```

>解析1： 如果常量 `WP_USE_THEMES` 存在且值为真，则判断页面类型同时给 $template 变量赋相应值；其中，判断页面类型的函数如 is_404() 位于`wp-includes` 目录下 `query.php` 文件，该函数返回对象 `$wp_query` 中 is_404() 方法，若 is_404() 为 false 则继续往下判断是否是其他页面；若为true 则给 $template 赋值为 get_404_template()，该函数位于 wp-includes 目录下 template.php 文件，它返回 `get_query_template('404')`，而该函数将页面类型传入数组 `$templates` 并应用调用函数 `locate_template($templates) `且应用过滤器；`locate_template()` 函数根据传入数组在主题中查找到相应的文件然后交给` load_template() `函数然后使用 require 加载，最终将用户需要的页面呈现出来；


