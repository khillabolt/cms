<!--
id: how-to
title: Customize
order: 4
-->

<div class="alert alert-info">
	<a href="%url%">Current site</a> it's an great example
	how to use <strong>Vestibulum CMS</strong>! Visit source on
	<a href="https://github.com/OzzyCzech/vestibulum/tree/master/public" target="_blank">GitHub</a>.
</div>

## Customize functions

### Add custom Twig functions/filters

Open `functions.php` in main folder and add your custom functions:

    function myUrl() {
      global $cms; /** @var \vestibulum\Vestibulum $cms */
      return $cms->url($_SERVER['REQUEST_URI']);
    }

In Twig template will be accessible `{{ myUrl() }}`

    function myFilter($string) {
      return ' ::: ' . $string . '  ::: ';
    }

In Twig template will be accessible `{{ title|myFilter }}`


### Change something in config

You can change your `config.php` or overwrite something directly from `functions.php`

    /** @var \vestibulum\Vestibulum $cms */
    $cms->config()->title = 'Vestibulum';
    $cms->config()->example = 'example';

In Twig template will be accessible `{{config.example}}`

## Hacking response

### JSON response

It's easy to overwrite main response. Just add `ajax.php` to your **src or current directory**:

    <?php
    if (!$this instanceof \vestibulum\Vestibulum) die('Sorry can be executed only from Vestibulum'); // protection

    // first detect AJAX Request
    if(!empty($_SERVER['HTTP_X_REQUESTED_WITH']) && strtolower($_SERVER['HTTP_X_REQUESTED_WITH']) == 'xmlhttprequest') {
      header('Content-Type: application/json');
      exit(json_encode(['will' => 'This will be my message']));
    }

You cen see example response here [%url%ajax](%url%ajax).

### Submitting HTML form

Lets have `contact.php` and `contact.html` in your **src directory**. Whole requires will be process in chain.
PHP goes first after that it's prepare HTML content:

    <?php
    if (!$this instanceof \vestibulum\Vestibulum) die('Sorry can be executed only from Vestibulum'); // protection

    if ($_SERVER['REQUEST_METHOD'] === 'POST') {
      @mail('info@example.com', 'My Subject', $_POST['message']); // send email...
      $this->flash = 'It's send well'; // can be used in Template file for example
    }

and HTML need contains

    <form method="post">
      <textarea name="message"></textarea>
      <button type="submit">Send</button>
    </form>

## Hacking template

### Executing Twig on content

It's simple Vestibulum support [template_from_string](http://twig.sensiolabs.org/doc/functions/template_from_string.html) function:

    {{ include(template_from_string(content)) }}

Now will be whole content processed with Twig parser. You can also parse only selected files:

    <!--
    twig: true
    -->

### Absolute image URL

Add follow code to your `functions.php` and all markdown images URL will be replaces with absolute URL:

    $cms->file->setContent(
    	preg_replace_callback(
    		'{(!\[.+\]\s?\()(\S*)([ \n]*(?:[\'"].*?[ \n]*[\'"])?\))}xsU', function ($matches) use ($cms) {
    			$path = $cms->file->isDir() ? $cms->file->getRealPath() : dirname($cms->file);
    			$path = str_replace(dirname($cms->src()), '', $path . '/');
    			return $matches[1] . $cms->url($path . $matches[2]) . $matches[3];
    		}, $cms->file->getContent()
    	)
    );


### Generate nested HTML menu

Vestibulum contains class `Pages`, it's smart helper for iterate over src files:

    <nav>
    	{% macro menu(pages, active, src) %}
    		<ul>
    			{% for page in pages %}
    				<li{% if active == page.id %} class="active"{% endif %} id="{{ page.id }}">
    					<a href="{{ url(page.slug(src)) }}">{{ page.title }}</a>
    					{% if page.children %}{{ _self.menu(page.children, active, src) }}{% endif %}
    				</li>
    			{% endfor %}
    		</ul>
    	{% endmacro %}
    	{{ _self.menu(pages, file.id, config.src) }}
    </nav>

And you also will need add follow code to your `functions.php`

    $cms->pages = Pages::from($cms->src())->toArraySorted();

## Hacking speed

### Replace Twig with plain phtml

Follow example is for those who have an performance obsession :-). First add `index.phtml` or `index.php` to your current working directory:

    <!DOCTYPE html>
    <html lang="en" ng-app="help">
    <head>
      <meta charset="utf-8">
      <title><?= $file->title ?> | <?= $config->title ?></title>
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <meta name="description" content="<?= $meta->description ?>">
      <meta name="robots" content="all"/>
      <meta name="author" content="<?= $meta->author ?>"/>
      <link rel="shortcut icon" href="<?= $this->url('favicon.ico') ?>" type="image/x-icon"/>
    </head>

    <div class="container">
      <? menu(); /* call your functions */ ?>
      <?= $content ?>
    </div>

    <link href="//netdna.bootstrapcdn.com/bootstrap/3.1.1/css/bootstrap.min.css" rel="stylesheet"/>
    <script src="//ajax.googleapis.com/ajax/libs/jquery/1/jquery.min.js"></script>
    <script src="//netdna.bootstrapcdn.com/bootstrap/3.1.1/js/bootstrap.min.js"></script>

    </body>
    </html>

Create markdown file, and change metadata:

    <!--
    template: index.phtml
    -->

This is it! Now can have request even **under 6 ms**

### Replace Composer autoloader

And one more things, you can save 20ms when you avoid Composer Autoloader

    require_once __DIR__ .'/../src/Vestibulum.php';
    require_once __DIR__ .'/../vendor/erusev/parsedown/Parsedown.php';
    require_once __DIR__ .'/../vendor/twig/twig/lib/Twig/Autoloader.php';
    Twig_Autoloader::register();
    echo new \vestibulum\Vestibulum; // deathly simple
