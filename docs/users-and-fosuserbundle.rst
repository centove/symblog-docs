[Part 7] - Users: Using the FOSUserBundle
=========================================

Overview
--------
At this point we now have a good start on our blog, and have accomplished quite a bit. We have installed Symfony2, learned a bit about how the core of Symfony works. Implemented some testing and got some sample data installed. Now we are going to look at adding some functionality that will allow us to actually start making use of the project. To do that we will need some sort of Users and a way to authenticate and authorize them do things on our blog.

Security in Symfony2
--------------------
Security in Symfony is handled by the security `component <https://github.com/symfony/Security>`_ it comes bundled with the `Symfony Standard Edition <http://symfony.com/download>`_ so there is nothing else to install from the framework point of view. The basic goal is we want to identify users, authenticate them, then authorize them to do specific actions on the our blog.
Symfony has excellent documentation on this in the `security <https://github.com/symfony/Security>`_ chapters in the book.

After looking through the documentation we see we could very well build our own custom user manager and deal with all the little details that go along with that. However the good folks that call themselves `FriendsOfSymfony <https://github.com/FriendsOfSymfony>`_ have done all this work already and made it available for us. All we need to do is install it and use it.


Install the FOSUserBundle
-------------------------
Looking around a bit we see that the `FOSUserBundle <https://github.com/FriendsOfSymfony/FOSUserBundle>`_ contains extensive `documentation <https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/Resources/doc/index.md>`_ on how to install and use the bundle. For the most part we will be following along with the exception that we will be installing from the a stable fixed point rather then the default tracking of the HEAD or master branch.

Installing this bundle, is similar to when we installed the doctrine-fixtures bundle, which consisted of:

    * Obtaining the source code for the bundle.
    * Registering the Namespace.
    * Any needed configuration.
    * Activate the bundle.

So we start off by consulting the documentation, and we see that one of the prerequisites is that the translator component needs to be enabled. So lets go ahead and do that.

.. code-block:: yaml

    # app/config/config.yml

    framework:
        translator: { fallback: en }

Okay so now we want to see where the bundle expects to live, typically this will be `vendor/bundles/<VENDOR>/<BUNDLE>` directory, and in this case we see it to be true. We also will be using the `bin/vendors` script to accomplish this. So lets look at that section. We see that it gives us what to add to the `deps` file to accomplish this.

.. code-block:: text

    [FOSUserBundle]
        git=git://github.com/FriendsOfSymfony/FOSUserBundle.git
        target=bundles/FOS/UserBundle 

We'll want to make a change to this as we want a specific version to be downloaded so instead of the above we'll add this.

.. code-block:: text

    [FOSUserBundle]
        git=git://github.com/FriendsOfSymfony/FOSUserBundle.git
        target=bundles/FOS/UserBundle
        version=1.1.0

Next we'll want to register the namespace for this bundle, again consulting the documentation we see that namespace is `FOS` so lets go ahead and add that to our `autoload.php`.

.. code-block:: php

    <?php
    // app/autoload.php

    $loader->registerNamespaces(array(
    // ...
        'FOS'               => __DIR__.'/../vendor/bundles',
    ));
    
Then we need to enable the bundle.

.. code-block:: php

    <?php
    // app/AppKernel.php

        public function registerBundles()
        {
            $bundles = array(
                // ...
                new FOS\UserBundle\FOSUserBundle(),
            );
        }
            
**Configuring the bundle**

Now as the goal of the bundle is to persist some `User` class in a database, this means we'll need an entity. At this point the one documented will work for what we want so lets go ahead and make that but have it in our bundle.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/User.php
    namespace Blogger\BlogBundle\Entity;

    use FOS\UserBundle\Entity\User as BaseUser;
    use Doctrine\ORM\Mapping as ORM;

    /**
     * @ORM\Entity(repositoryClass="Blogger\BlogBundle\Repository\UserRepository")
     * @ORM\Table(name="blog_user")
     */
    class User extends BaseUser
    {
        /**
        * @ORM\Id
        * @ORM\Column(type="integer")
        * @ORM\GeneratedValue(strategy="AUTO")
        */
        protected $id;
    
        public function __construct()
        {
            parent::__construct();
        
        }
    }
    

.. note::

    'User' is a reserved keyword in SQL, so you must name your table something else.

Now we want to tell Symfony's security component to use this bundle, so we'll update our `security.yml` file to reflect that. Again the `default` in the bundle's documentation will work for our needs so we'll go ahead and use that.

.. code-block:: yaml

    # app/config/security.yml
    security:
        providers:
            fos_userbundle:
                id: fos_user.user_manager

        encoders:
            "FOS\UserBundle\Model\UserInterface": sha512

        firewalls:
            main:
                pattern: ^/
                form_login:
                    provider: fos_userbundle
                logout:       true
                anonymous:    true

        access_control:
            - { path: ^/login$, role: IS_AUTHENTICATED_ANONYMOUSLY }
            - { path: ^/register, role: IS_AUTHENTICATED_ANONYMOUSLY }
            - { path: ^/resetting, role: IS_AUTHENTICATED_ANONYMOUSLY }
            - { path: ^/admin/, role: ROLE_ADMIN }

        role_hierarchy:
            ROLE_ADMIN:       ROLE_USER
            ROLE_SUPER_ADMIN: ROLE_ADMIN

At this point, our blog is now set up to use the `FOSUserBundle` however we have one final bit of configuration to accomplish and that deals with configuring the bundle itself. At this point we will do the minimal config to get things working, though we will be coming back to add/tweak the configuration as we get further along in our development. So lets go ahead and give it the minimal config. Also the documentation shows the configuration taking place in `app/config/config.yml` which is fine if you want to do it there. I prefer to keep the configuration for the third party bundles I install segmented out so that they are easier for me to find and update. As such we do a bit more to get things working but save time in the future.

In the config.yml we add this.

.. code-block:: yaml

    # app/config/config.yml
    imports:
       # ...
       - { resource: fos_user.yml }


Then we create the `app/config/fos_user.yml` file and place this in it.

.. code-block:: yaml

    # app/config/fos_user.yml
    fos_user:
        db_driver: orm 
        firewall_name: main
        user_class: Blogger\BlogBundle\Entity\User

Now we can import the routing files so that we get the routes for the functionality this bundle is bringing us. You can place them either in the `app/config/routing.yml` file or in the `src/Blogger/BlogBundle/Resources/config/routing.yml` file. I prefer keeping things together so let's put it in `src/Blogger/BlogBundle/Resources/config/routing.yml`

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    fos_user_security:
        resource: "@FOSUserBundle/Resources/config/routing/security.xml"

    fos_user_profile:
        resource: "@FOSUserBundle/Resources/config/routing/profile.xml"
        prefix: /profile

    fos_user_register:
        resource: "@FOSUserBundle/Resources/config/routing/registration.xml"
        prefix: /register

    fos_user_resetting:
        resource: "@FOSUserBundle/Resources/config/routing/resetting.xml"
        prefix: /resetting

    fos_user_change_password:
        resource: "@FOSUserBundle/Resources/config/routing/change_password.xml"
        prefix: /profile

As you can see by the above, it looks like we will be getting a good chunk of functionality with this bundle. So lets go ahead and get it installed.

.. code-block:: bash

    $ bin/vendors install

Once that completes, you can see by running a few commands that the bundle is active and almost ready to use.

.. code-block:: bash

    $ php app/console route:debug
    
You should see a number of routes `fos_user_xxxxx` in the list now.

.. code-block:: bash

    $ php app/console
    
Will show some new commands under the fos name.

Now that the bundle is installed, configured and activate and it's main purpose is to persist a `User` entity to the database we need to update the database. Lets go ahead and do that.

.. code-block:: bash

    $ php app/console doctrine:migrations:diff
    $ php app/console doctrine:migrations:migrate
    
Now looking at the database we should see a 'blog_user' table. We can check that things are really working by visiting some of the following links.

To login ``http://symblog.dev/app_dev.php/login``.
To register a user ``http://symblog.dev/app_dev.php/register``.

And so on, you'll notice however things don't look quite right and all our nice css and layout vanished. Don't fret we'll deal with that next.




* Fixtures
  Now that we have a User entity we'll need a few fixtures for it, as we are using a bundle for our User we have to take some special care to add the users to the database. While a normal fixture loader will get the information in the database the actual entries won't work as the UserManager isn't used to actually create the user. So we use the ContainerAware fixture loader as thus:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/DataFixtures/ORM/UserFixtures.php
    namespace Blogger\BlogBundle\DataFixtures\ORM;

    use Doctrine\Common\DataFixtures\AbstractFixture;
    use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
    use Symfony\Component\DependencyInjection\ContainerAwareInterface;
    use Symfony\Component\DependencyInjection\ContainerInterface;

    use Blogger\BlogBundle\Entity\User;

    class LoadUserData extends AbstractFixture implements OrderedFixtureInterface, ContainerAwareInterface
    {
        private $container;
        public function setContainer(ContainerInterface $container = null)
        {
            $this->container = $container;
        }

        public function load(\Doctrine\Common\Persistence\ObjectManager $manager)
        {
            $userManager = $this->container->get('fos_user.user_manager');
            $user = $userManager->createUser();
            $user->setUsername('admin');
            $user->setEmail('me@example.com');
            $user->setPlainPassword('NoPass4U!');
            $user->setEnabled(true);
            $user->addRole('ROLE_ADMIN');
            $userManager->updateUser($user);
    
            $manager->persist($user);
            $manager->flush();
            $this->addReference('admin-user', $user);
        }
    
        public function getOrder()
        {
        return 1;
        }
    }

* Update views to include the fos_views
* doctrine-migrate

Adding a field to the User Entity
---------------------------------

* doctrine-migrate

Override the bundle resources
-----------------------------
* Forms
* Views
* Translations

Profile Editing/Password reset links
------------------------------------


Roles
-----

Blog Entity Forms/User integration
-----------------------------------

Comment Moderations
-------------------

Testing
-------


