title: wordpress删除googleapi
date: 2016-10-12 22:23:20
tags: wordpress
---

## wordpress加载慢

wordpress默认启用twentysixteen皮肤，注释掉`wp-content/theme/twentysixteen/functions.php`中的代码：

wordpress很有趣，今年是2016年，那么默认皮肤就是2016...
```php
	function twentysixteen_scripts() {
	// Add custom fonts, used in the main stylesheet.
	//注释掉这段code即可
	//wp_enqueue_style( 'twentysixteen-fonts', twentysixteen_fonts_url(), array(), null );

	// Add Genericons, used in the main stylesheet.
	wp_enqueue_style( 'genericons', get_template_directory_uri() . '/genericons/genericons.css', array(), '3.4.1' );
```