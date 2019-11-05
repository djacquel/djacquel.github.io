---
title: "Symfony CMS - day 02 - Behavior Driven Development (BDD) with Behat and Mink"
comments: true
---

How to do Behavior Driven Development (BDD) with Symfony

Second day of our [Build CMS Application with Symfony][cms0] articles suite.

Previous article [day 01 - Symfony installation][cms1].

## Usefull readings

[Behat documentation][Behat]

## Install

We'll install :

  - our BDD tools : [phpunit][phpunit], [Behat symfony extension][Behat symfony extension], [Mink extension][Mink extension] and browserkit.
  - fixtures tools to be able to load test datas : [Doctrine fixtures Bundle][Doctrine fixtures Bundle] and [Behat Fixtures extension][Behat Fixtures extension].

In **composer.json** file add :
{% highlight json %}
[...]
    "require": {
        [...],
        "doctrine/data-fixtures": "1.0.*@dev",
        "doctrine/doctrine-fixtures-bundle": "dev-master",
        "behat/symfony2-extension": "*",
        "behat/mink-extension": "*",
        "behat/mink-browserkit-driver": "*",
        "vipsoft/doctrine-data-fixtures-extension": "*"
    },
[...]
{% endhighlight %}

Run a
{% highlight bash %}
composer update
{% endhighlight %}

Enable Fixture Bundle in app/AppKernel.php
{% highlight php %}
class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = array(
          [...]
          new Doctrine\Bundle\FixturesBundle\DoctrineFixturesBundle(),
[...]
{% endhighlight %}

You now have a full BDD tool onboard, we'll configure it.

### Creating a test bundle

In order to have all our BDD tests at the same place, we create an empty bundle.
We choose "annotation" for config format and no route update.

{% highlight bash %}
app/console generate:bundle --namespace=My/BDDBundle
{% endhighlight %}

Ensure that your bundle is enabled in app/AppKernel.php.

{% highlight php %}
class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = array(
          [...]
          new My\BDDBundle\MyBDDBundle()
[...]
{% endhighlight %}

We then create folder to host our BDD tests in our BDD Bundle.
{% highlight bash %}
bin/behat --init "@MyBDDBundle"
{% endhighlight %}

## Using Behat

### Configure Behat

Create a **behat.yml** file at your site root.

{% highlight yaml %}
# behat.yml
default:
    paths:
        features: src/My/BDDBundle/Features/
        bootstrap: %behat.paths.features%/bootstrap
    context:
        class:  My\BDDBundle\Features\Context\FeatureContext

    extensions:
        Behat\Symfony2Extension\Extension:
            mink_driver: true

        Behat\MinkExtension\Extension:
            base_url:  'http://tuto.dev/app_dev.php'
            default_session: 'symfony2'

        # will use this in a next article
        #VIPSoft\DoctrineDataFixturesExtension\Extension:
        #  lifetime: feature
        #  autoload: true
        #  fixtures: ~
{% endhighlight %}

## Create your first feature test

In **src/My/BDDBundle/Features/** create a file named **01-symfony-install.feature**. [More about features on behat.org][More about features on behat.org].

{% highlight yaml %}
# src/My/BDDBundle/Features/01-symfony-install.feature
Feature: Fresh Symfony installation
  In order to start developing a Symfony application
  As a smart developer
  I need to be able to see the default Symfony Standard Edition Home page

  Scenario: View the home page
    Given I am on the homepage
    Then I should see "I love Espelette pepper !"
{% endhighlight %}

Run a
{% highlight bash %}
bin/behat
{% endhighlight %}

This is supposed to give you a nice

{% highlight bash %}
Feature: Fresh Symfony installation
  In order to start developing a Symfony application
  As a smart developer
  I need to be able to see the default Symfony Standard Edition Home page

  Scenario: View the home page
    Given I am on the homepage
    Then I should see "I love Espelette pepper !"

1 scenario (1 undefined)
2 steps (2 undefined)
0m0.041s

You can implement step definitions for undefined steps with these snippets:

    /**
     * @Given /^I am on the homepage$/
     */
    public function iAmOnTheHomepage()
    {
        throw new PendingException();
    }

    /**
     * @Then /^I should see "([^"]*)"$/
     */
    public function iShouldSee($arg1)
    {
        throw new PendingException();
    }
{% endhighlight %}

Behat is unable to find steps ([steps documentation on behat.org][steps documentation on behat.org]) and advise you to implement them.

But wait ! We have **Mink** onboard ! And Mink comes with predefined steps.

Open your **src/My/BDDBundle/Features/Context/FeatureContext.php** and make your class extend **MinkContext**.

{% highlight php %}
<?php

  namespace My\BDDBundle\Features\Context;

  use Behat\Symfony2Extension\Context\KernelAwareInterface,
      Symfony\Component\HttpKernel\KernelInterface;

  use Behat\Behat\Exception\PendingException;
  use Behat\MinkExtension\Context\MinkContext;

  /**
   * Feature context.
   */
  class FeatureContext extends MinkContext
                    implements KernelAwareInterface
  {
    ...
  }
  ?>
{% endhighlight %}

To see the list of all predefined steps in Mink, just :
{% highlight bash %}
bin/behat -dl
#or
bin/behat -di
{% endhighlight %}

ReRun a
{% highlight bash %}
bin/behat
{% endhighlight %}

And now you get :

{% highlight bash %}
Feature: Fresh Symfony installation
  In order to start developing a Symfony application
  As a smart developer
  I need to be able to see the default Symfony Standard Edition Home page

  Scenario: View the home page
    Given I am on the homepage
    Then I should see "I love Espelette pepper !"
      The text "I love Espelette pepper !" was not found anywhere in the text of the current page.

1 scenario (1 failed)
2 steps (1 passed, 1 failed)
0m0.124s
{% endhighlight %}

Oups ! We have an error ! We can't see "I love Espelette pepper !".
Ok, this is the normal way for testing : write tests, run test that fail, develop then run tests that succeed.

Edit src/Acme/DemoBundle/Resources/views/Welcome/index.html.twig and change

{% highlight html %}
<h1 class="title">Welcome!</h1>
for
<h1 class="title">I love Espelette pepper !</h1>
{% endhighlight %}

ReReRun a
{% highlight bash %}
bin/behat
{% endhighlight %}

All the lights are now green !

Don't forget to [read behat doc][read behat doc] and about [Espelette pepper][Espelette pepper].

Next article will be on [Sonata Admin Installation][cms3].

See you soon.

{% include _links_library.markdown %}
