 In `my-plugin/my-plugin.php`:
 
 ```php
 public function onPluginsInitialized(): void {
    ...

   $this->router();
 }
public function router() {
  /** @var Uri $uri */
  $uri = $this->grav['uri'];
  $route = Uri::getCurrentRoute()->getRoute();

  if (Utils::startsWith($route, '/locations')) {
    $this->enable([
      'onPagesInitialized' => ['addLocationPage', 0]
    ]);
  }
}
public function addLocationPage() {
  $route = Uri::getCurrentRoute()->getRoute();
  $parts = explode("/", $route);
  $path = array_shift($parts);

  /** @var Pages $pages */
  $pages = $this->grav['pages'];

  if ($pages->find($route)) {
    /** @var Debugger $debugger */
    $debugger = $this->grav['debugger'];
    $debugger->addMessage("Page {$route} already exists, page cannot be added", 'error');
    return;
  }

  $flex = Grav::instance()->get('flex');
  $location = $flex->getObject($path, 'locations');

  $page = $pages->find('/locations/location');
  if ($page) {
    $page->id($page->modified() . md5($route));
    $page->slug(basename($route));
    $page->folder(basename($route));
    $page->route($route);
    // $page->rawRoute($route);
    $page->modifyHeader('object', $path);
    if ($location) {
      $page->modifyHeader('title', $location->getProperty('name'));
    }
    $pages->addPage($page, $route);
  }
}
```
 
that's all the code for it in my-plugin/my-plugin.php
then i created a placeholder page at /locations/location/location.md with just this for front matter 
```yaml
---
title: Location
object: null
---
```
So basically what's happening is upon visiting that route, your plugin is modifying the barebones page on the fly and replacing object: null with your flex object path.
Then in my template for that page, location.html.twig i use that path that was replaced to retrieve the flex object and use it in my template like so
```twig
{% set flex = grav['flex_objects'] %}
{% set directory = flex.directory('locations') %}
{% set object = directory.getObject(header.object) %}

<h1 class="text-3xl md:text-5xl">
  {{ object.name }}
</h1>
```
