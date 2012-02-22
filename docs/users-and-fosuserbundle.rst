[Part 7] - Users: Using the FOSUserBundle
=========================================

Overview
--------

Security in Symfony2
--------------------

Install the FOSUserBundle
-------------------------

.. tip::
  We are using the 'stable' branch of the FOSUserBundle.
* Installing
* Configuring
* Entity

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
            $user->setName('dsyph3r');
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


