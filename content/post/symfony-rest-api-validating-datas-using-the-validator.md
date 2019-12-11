---
title:  "Symfony REST api : validating data that will be stored in different entities"
description: "How to solve the validation problem when designing a REST api with Symfony"
date:   2016-06-16T14:49:34+02:00
author: Sébastien Le Gall
categories: [php,symfony]
---

REST is great. But, most of the time, it is quite hard to think your web services architecture in a way that each WS is stateless. Let's think about a common use case.

## The problem
If you have to design a personal data form and wish to ask the user to complete some fields such as the `name`, the `firstname`, the `birthdate`, and two addresses with, for each, an `addresse`, a `city`, a `zipcode`, and a `country`.

Using Symfony and Doctrine, you will probably have to create 3 entities :

```php

<?php
class User {
  protected $id;
  protected $name;
  protected $firstname;
  protected $birthdate;

  // getters and setters
}

class Address {
  protected $id;
  protected $address;
  protected $city;
  protected $zipcode;
  protected $country;

  // getters and setters
}

class UserAddress {
  protected $user;
  protected $address;

  // getters and setters
}

?>

```
In that case, your database model will probably looks like that :

| user  | user_address | address |
|---|---|---|
|  id_user |  id_user |  id_adress |
|  name |  id_address |  address |
|  firstname |  |  city |
|  birthdate |  |  zipcode |
|   |   |  country |

Now, if you wish to strictly use REST standards, you would expose 2 WS :

 * One that POST a user
 * One that POST an address (using user id)

 And then, for each, populate your objects, check some rules such as the name is only composed of letters, using the Validator Symfony Component.

```php
<?php
$user = new User();
$user->setName($request->get('name'));

$errors = $this->validator->validate($user);
if($errors) {
  //...
}
//...
$user->persist();
$em->flush();

return new JsonResponse($user->id, 201);
?>
```

```php
<?php
$address = new Address();
$address->setAddress($request->get('address'));

$errors = $this->validator->validate($address);
if($errors) {
  //...
}

//...
$user = $this->get('em')->getRepository()->findById($request->get('user_id'));
$userAddress = new UserAddress();
$userAddress->setUser($user);
$userAddress->setAddress($address);

$userAddress->persist();
$address->persist();

$em->flush();

?>
```

However, most of the time, this is probably not what you'll do. There are some reasons why :

* Making at least 3 calls to post a simple form could be a bad idea if you wish to improve performances
* That way, it will be quite difficult to check, for example, that the 2 required addresses are different. (You will probably have to post the first one and then check the DB to compare with the second one)
* Some of your WS will be stateless but won't make any sens as a domain.

A common way to avoid such problems is using the Symfony Form Builder. Then, you need to populate your form with the request data and test the Symfony validator method `isValid()` on the form.

**This is not a good way to deal with REST data validation.**

* First of all, Symfony forms are design to actually generate HTML form.
* Why would you create a form object (using a factory) when you already have a form on the front app?
* Form validation is great when you need to actually show (html way) errors. If your goal is to send an error message to your frontend app... well... that's not the purpose.

## My solution
I have just released [a little bundle](https://github.com/seblegall/api-validator-bundle) that provide a good solution to answer that kind of dilemma.

Instead of exposing 2 WS, we will only expose one. we will be able to send all datas as a Json object and validate the Json object instead of validate each domain object apart.

Here is what the Json object could looks like in our example :

```json
{
   "name":"Doe",
   "firstname":"John",
   "birthdate":"11/24/1988",
   "addresses":[
      {
         "address":"36, Disney Road",
         "city":"Paradize",
         "zipcode":"764576",
         "country":"France"
      },
      {
         "address":"37, Mickey Road",
         "city":"Paradize",
         "zipcode":"764576",
         "country":"France"
      }
   ]
}
```

Then, we will validate all that object using the Symfony Parameter Bag and the Validator Component.

Let's try.

### Installation

Using composer :

```sh
$ composer require seblegall/api-validator-bundle
```

Then, set up the bundle in your Symfony's `AppKernel.php` file :

```php
<?php
class AppKernel extends Kernel
{
    public function registerBundles()
    {
        return array(
            new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
            // ...
            new Seblegall\ApiValidatorBundle\ApiValidatorBundle(),
        );
    }
// ....
?>
```
Finally, import the bundle routing by inserting those lines in the `app/config/routing.yml` file :

```yml
api_validator_bundle:
    resource: "@ApiValidatorBundle/Resources/config/routing.yml"
```

Once the bundle is installed, here is how to do so.

### Create a dedicated Parameter Bag for your web service

* Create a directory in the bundle structure. The directory can be named as you wish. For example `Api` or `ParameterBag`.

You now should have something like that :

```sh
src\
  MyBundle\
    Api\
    Controller\
    DependencyInjection\
    Resources\
    MyBundle.php
```

* Create a PHP Class that extends `Seblegall\ApiValidatorBundle\Request\ApiParameterBag` :

```php
<?php

namespace MyBundles\Api;

use Seblegall\ApiValidatorBundle\Request\ApiParameterBag;

class ExampleApiParameterBag extends ApiParameterBag
{

    // return the parameters you need to catch.
    // It could be  :
    //static::PARAMETERS_TYPE_HEADERS
    //static::PARAMETERS_TYPE_QUERY
    //static::PARAMETERS_TYPE_REQUEST
    //static::PARAMETERS_TYPE_COOKIES
    //static::PARAMETERS_TYPE_FILES
    //static::PARAMETERS_TYPE_ATTRIBUTES

    public function getFilteredType()
    {
        return array(
            static::PARAMETERS_TYPE_QUERY,
            static::PARAMETERS_TYPE_REQUEST,
        );
    }

    // return the parameters key you need to catch.
    public function getFilteredKeys()
    {
        return array('firstname', 'name', 'birthdate');
    }
}
?>
```

### Enable your new parameter bag.

Using yml, in the bundle `routing.yml` file :

```yml
example_api:
    path:     /example/api/ws-route
    defaults: { _controller: MyBundle:Api:ws , _api_bag: MyBundle\Api\ExampleApiParameterBag }
```
Or, if you prefer, using annotations, directly in the controller :

```php
<?php
/**
 * @ApiParameters(bag="MyBundle\Api\ExampleApiParameterBag")
 *
 * @param Request $request
 *
 * @return JsonResponse
 */
public function ws2Action(Request $request)
{
    $apiParameters = $request->attributes->get('api_parameters');

    return new JsonResponse($apiParameters->all());
}
?>
```

### Add validation rules

If not yet create, add a `validation.yml` file in the `Resources` directory.

Add your own validation rules. Here is an example :

```yml
MyBundle\Api\ExampleApiParameterBag:
    properties:
        parameters:
            - Collection:
                fields:
                    firstname:
                        - Type:
                            type: string
                    name:
                        - Type:
                            type: string
                    birthdate:
                        - Type:
                            type: datetime # This could be a custom constraint
                allowMissingFields: true
                allowExtraFields: true

```

### Enable validation

Using the `routing.yml` file :

```yml
example3_api:
    path:     /example/api/ws-route-validation
    defaults: { _controller: MyBundle:Api:ws , _api_bag:MyBundle\Api\ExampleApiParameterBag, _api_validation: true }
```

Or, using annotations :

```php
<?php
/**
 * @ApiParameters(bag="MyBundle\Api\ExampleApiParameterBag", validation=true)
 *
 * @param Request $request
 *
 * @return JsonResponse
 */
public function ws3Action(Request $request)
{
    $apiParameters = $request->attributes->get('api_parameters');

    return new JsonResponse($apiParameters->all());
}
?>
```

### You're done

Now, any request send to your web service will be catch and the request data will be validate before calling the controller in a way that you don't need to validate anything in the controller or in any service one the controller is called.

### What if datas are not valid ?

If datas are not valid, the bundle will return a status code `400` and an array containing an `errors` key in which you will find all the errors catch by the validator component.

## Learn More

[See the github project](https://github.com/seblegall/api-validator-bundle).
