Block Types
===========

For each purpose a different block type can be used. The general purpose
blocks can be used for several solutions.  The Block Bundle ships with more
specific block types.

RssBlock
--------

The RssBlock extends the ActionBlock and allows you to read feed items and
display them in a list.

Create a document::

    use Symfony\Cmf\Bundle\BlockBundle\Document\RssBlock;

    // ...

    $myRssBlock = new RssBlock();
    $myRssBlock->setParentDocument($parentPage);
    $myRssBlock->setName('rssBlock');
    $myRssBlock->setSetting('title', 'Symfony2 CMF news');
    $myRssBlock->setSetting('url', 'http://cmf.symfony.com/news.rss');
    $myRssBlock->setSetting('maxItems', 3);

    $documentManager->persist($myRssBlock);

All available settings are:

* **url**: the url of the rss feed (*required*)
* **title**: the title for the list (*default*: Insert the rss title)
* **maxItems**: the maximum amount of items to return to the template
  (*default*: 10)
* **template**: the template to render the feed items (*default*:
  ``CmfBlockBundle:Block:block_rss.html.twig``)
* **ItemClass**: the class used for the item objects that are passed to the
  template (*default*: ``Symfony\Cmf\Bundle\BlockBundle\Model\FeedItem``)

The controller to get the feed items can also be changed:

* Define a different class for the controller service in your configuration
  using the DI service parameter ``cmf_block.rss_controller_class``
* or set the actionName of your RssBlock document

.. note::

        The `Symfony CMF Sandbox`_ contains an example of the RssBlock.

ImagineBlock
------------

The imagine block uses the `LiipImagineBundle`_ to display images directly
out of PHPCR. The block has a child of type ``nt:file`` and fields for the
name of the imagine filter to use, an URL and an image caption. To use this
block, you need to add ``liip/imagine-bundle`` to your composer.json and
define the imagine filter you specify in the block. The default name is
``cmf_block``. The filter must use the ``phpcr`` driver:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        liip_imagine:
            # ...
            filter_sets:
                cmf_block:
                    data_loader: phpcr
                    quality: 85
                    filters:
                        thumbnail: { size: [616, 419], mode: outbound }
                # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services">

            <config xmlns="http://example.org/dic/schema/liip_imagine">
                <!-- ... -->
                <filter-set name="cmf_block" data-loader="phpcr" quality="85">
                    <filter name="thumbnail" size="616,419" mode="outbound"/>
                </filter-set>
                <!-- ... -->
            </config>
        </container>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('liip_imagine', array(
            // ...
            'filter_sets' => array(
                'cmf_block' => array(
                    'data_loader' => 'phpcr',
                    'quality'     => 85,
                    'filters'     => array(
                        'thumbnail' => array(
                            'size' => array(616, 419),
                            'mode' => 'outbound',
                        ),
                    ),
                ),
                // ...
            ),
        ));

Refer to the `LiipImagineBundle documentation`_ for further information.

See the example below for how to create an ``ImagineBlock`` programmatically.

SlideshowBlock
--------------

The ``SlideshowBlock`` is just a special kind of ``ContainerBlock``. It
can contain any kind of blocks that will be rendered with a wrapper div
to help a javascript slideshow library to slide them.
The ``ImagineBlock`` is particularly suited if you want to do an image
slideshow but the ``SlideshowBlock`` can handle any kind of blocks, also mixed
types of blocks in the same slideshow.

.. note::

    This bundle does not attempt to provide a javascript library for animating
    the slideshow. Chose your preferred library that plays well with the rest
    of your site and hook it on the slideshows. (See also below).


Create your first slideshow
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Creating a slideshow consists of creating the container ``SlideshowBlock`` and
adding blocks to it. Those blocks can be anything, but an image makes a lot
of sense::

    use Symfony\Cmf\Bundle\BlockBundle\Document\SlideshowBlock;
    use Symfony\Cmf\Bundle\BlockBundle\Document\ImagineBlock;
    // the Image will be moved to Symfony\Cmf\Bundle\MediaBundle\Model\Image
    use Doctrine\ODM\PHPCR\Document\Image;
    use Doctrine\ODM\PHPCR\Document\File;

    // create slideshow
    $mySlideshow = new SlideshowBlock();
    $mySlideshow->setName('slideshow');
    $mySlideshow->setParentDocument($parentPage);
    $mySlideshow->setTitle('My first Slideshow');
    $documentManager->persist($mySlideshow);

    // add first slide to slideshow
    $mySlideshowItem = new ImagineBlock();
    $mySlideshowItem->setName('first_item');
    $mySlideshowItem->setLabel('label of first item');
    $mySlideshowItem->setParentDocument($mySlideshow);
    $manager->persist($mySlideshowItem);

    $file = new File();
    $file->setFileContentFromFilesystem('path/to/my/image.jpg');
    $image = new Image();
    $image->setFile($file);
    $mySlideshowItem->setImage($image);


Render the slideshow
~~~~~~~~~~~~~~~~~~~~

Rendering your slideshow is as easy as just rendering the according block
in your template. If your ``contentDocument`` has a field ``slideshow`` that
contains a ``SlideshowBlock`` object, you can simply render it with:

.. code-block:: jinja

    {{ sonata_block_render({
        'name': 'slideshow'
    }) }}


Make the slideshow work in the frontend
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Since the BlockBundle doesn't contain anything to make the slideshow work
in the frontend, you need to do this yourself. Just use your favourite JS
library to make the slideshow interactive. If special markup is needed for
your slideshow code to work, just override
``BlockBundle:Block:block_slideshow.html.twig`` or the templates of the
blocks you use as slideshow items and adapt them to your needs.


Use the Sonata admin class
~~~~~~~~~~~~~~~~~~~~~~~~~~

The BlockBundle comes with an admin class for managing slideshow blocks. All
you need to do to administrate slideshows in your project is to add the
following line to your sonata admin configuration:

.. configuration-block::

    .. code-block:: yaml

        sonata_admin:
            dashboard:
                groups:
                    blocks:
                        label: Blocks
                        items:
                            - cmf_block.slideshow_admin

However, you can also embed the slideshow administration directly into
other admin classes using the ``sonata_type_admin`` form type. The admin
service to use in that case is ``cmf_block.slideshow_admin``.
Please refer to the `Sonata Admin documentation`_
for further information.

.. _`Symfony CMF Sandbox`: https://github.com/symfony-cmf/cmf-sandbox
.. _`Sonata Admin documentation`: http://sonata-project.org/bundles/admin/master/doc/reference/form_types.html
.. _`LiipImagineBundle`: https://github.com/liip/LiipImagineBundle
.. _`LiipImagineBundle documentation`: https://github.com/liip/LiipImagineBundle/tree/master/Resources/doc
