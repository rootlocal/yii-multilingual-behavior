yii-multilingual-behavior
=========================

This behavior allow you to create multilingual models and to use them
(almost) like normal models. For each model, translations have to be
stored in a separate table of the database (ex: PostLang or
ProductLang), which allow you to easily add or remove a language without
modifying your database.

First example: by default translations of current language are inserted
in the model as normal attributes.

```php
// Assuming current language is english (in protected/config/main.php :
'sourceLanguage' => 'en')
$model = Post::model()->findByPk((int) $id);
echo $model->title; //echo "English title"
 
//Now let's imagine current language is french (in
protected/config/main.php : 'sourceLanguage' => 'fr')
$model = Post::model()->findByPk((int) $id);
echo $model->title; //echo "Titre en Français"
 
$model = Post::model()->localized('en')->findByPk((int) $id);
echo $model->title; //echo "English title"
 
//Here current language is still french
```

Second example: if you use multilang() in a "find" query, every
translation of the model are loaded as virtual attributes (title_en,
title_fr, title_de, ...).

```php
$model = Post::model()->multilang()->findByPk((int) $id);
echo $model->title_en; //echo "English title"
echo $model->title_fr; //echo "Titre en Français"
```

Requirements
------------


```sql
CREATE TABLE IF NOT EXISTS `postLang` (
`l_id` int(11) NOT NULL AUTO_INCREMENT,
`post_id` int(11) NOT NULL,
`lang_id` varchar(6) NOT NULL,
`l_title` varchar(255) NOT NULL,
`l_content` TEXT NOT NULL,
PRIMARY KEY (`l_id`),
KEY `post_id` (`post_id`),
KEY `lang_id` (`lang_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8;

ALTER TABLE `postLang`
ADD CONSTRAINT `postlang_ibfk_1` FOREIGN KEY (`post_id`) REFERENCES
`post` (`id`) ON
DELETE CASCADE ON UPDATE CASCADE;
```

Attach this behavior to the model (Post in the example). Everything that
is commented is with default values.




```php
public function behaviors() {
    return array(
        'ml' => array(
            'class' => 'application.models.behaviors.MultilingualBehavior',
            //'langClassName' => 'PostLang',
            //'langTableName' => 'postLang',
            //'langForeignKey' => 'post_id',
            //'langField' => 'lang_id',
            'localizedAttributes' => array('title', 'content'), //attributes of the model to be translated
            //'localizedPrefix' => 'l_',
            'languages' => Yii::app()->params['translatedLanguages'], // array of your translated languages. Example : array('fr' => 'Français', 'en' => 'English')
            'defaultLanguage' => Yii::app()->params['defaultLanguage'], //your main language. Example : 'fr'
            //'createScenarios' => array('insert'),
            //'localizedRelation' => 'i18nPost',
            //'multilangRelation' => 'multilangPost',
            //'forceOverwrite' => false,
            //'forceDelete' => true, 
            //'dynamicLangClass' => true, //Set to true if you don't want to create a 'PostLang.php' in your models folder
        ),
    );
}
```




Look into the behavior source code and read the comments of each
attribute to understand how to configure them and how to use the
behavior.

In order to retrieve translated models by default, add this function in
the model class:


```php
public function loadModel($id, $ml=false) {
    if ($ml) {
        $model = Post::model()->multilang()->findByPk((int) $id);
    } else {
        $model = Post::model()->findByPk((int) $id);
    }
    if ($model === null)
        throw new CHttpException(404, 'The requested post does not exist.');
    return $model;
}
```


and use it like this in the update action :



```php
public function actionUpdate($id) {
    $model = $this->loadModel($id, true);
    ...
}
```

Here is a very simple example for the form view : 


```php
<?php foreach (Yii::app()->params['translatedLanguages'] as $l => $lang) :
    if($l === Yii::app()->params['defaultLanguage']) $suffix = '';
    else $suffix = '_'.$l;
    ?>
<fieldset>
    <legend><?php echo $lang; ?></legend>
 
    <div class="row">
    <?php echo $form->labelEx($model,'title'); ?>
    <?php echo $form->textField($model,'title'.$suffix,array('size'=>60,'maxlength'=>255)); ?>
    <?php echo $form->error($model,'title'.$suffix); ?>
    </div>
 
    <div class="row">
    <?php echo $form->labelEx($model,'content'); ?>
    <?php echo $form->textArea($model,'content'.$suffix); ?>
    <?php echo $form->error($model,'content'.$suffix); ?>
    </div>
</fieldset>
<?php endforeach; ?>
```


To enable search on translated fields, you can modify the search()
function in the model like this :


```php
public function search()
{
    $criteria=new CDbCriteria;
 
    //...
    //here your criteria definition
    //...
 
    return new CActiveDataProvider($this, array(
        'criteria'=>$this->ml->modifySearchCriteria($criteria),
        //instead of
        //'criteria'=>$criteria,
    ));
}
```

Warning: the modification of the search criteria is based on a simple
str_replace so it may not work properly under certain circumstances.

It's also possible to retrieve languages translation of two or more
related models in one query. Example for a Page model with a "articles"
HAS_MANY relation : 


```php
$model = Page::model()->multilang()->with('articles', 'articles.multilangArticle')->findByPk((int) $id);
echo $model->articles[0]->content_en;
```

gt
[GitHub](http://github.com)

With this method it's possible to make multi model forms like it's
explained [here](http://www.yiiframework.com/wiki/19/how-to-use-a-single-form-to-collect-data-for-two-or-more-models/)

History
-------

**24/03/2012:** First release

**28/03/2012:** It's now possible to modify language when retrieving
data with the localized relation.

Example:


```php
$model = Post::model()->localized('en')->findByPk((int) $id);
```

**30/03/2012:** Modification of the rules definition for translated
attributes:

* if you set forceOverwrite to true, every rules defined in the model
  for the attributes to translate will be applied to the translations.
* if you set forceOverwrite to false (default), every rules defined in
  the model for the attributes to translate will be applied to the
  translations except "required" rules that will only be applied to the
  default translation.


**28/06/2012:** (Bug Fix)
[two2wyes](http://www.yiiframework.com/forum/index.php/topic/4888-multilingual-models/page__view__findpost__p__155755) found and fixed a bug that prevented translations to be correctly saved on attributes that only have a "required" rule and with the "forceOverwrite" option set to false. Thanks again to him. [See the thread](http://www.yiiframework.com/forum/index.php/topic/4888-multilingual-models/page__view__findpost__p__155755)


Resources
---------

* [Original
  Thread](http://www.yiiframework.com/forum/index.php/topic/4888-multilingual-models/)



Authors
-------

Many thanks to [guillemc](http://www.yiiframework.com/user/2677) who made the biggest part of the work on this
behavior ([see original
thread](http://www.yiiframework.com/forum/index.php/topic/4888-multilingual-models/)).



