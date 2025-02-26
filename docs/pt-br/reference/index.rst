:source_url: https://github.com/doctrine/persistence/blob/4.0.x/docs/en/reference/index.rst
:revision: 7422aabf621f2d246e27cd7474a570c11f55da2e
:status: ready

:title: Introdução

Introdução
==========

O projeto Doctrine Persistence é um conjunto de interfaces e funcionalidades
compartilhadas que os diferentes mapeadores de objetos Doctrine compartilham.
Você pode usar essas interfaces e classes abstratas para construir seu próprio
mapeador se não quiser usar os mapeadores de dados completos fornecidos pelo
Doctrine.

Instalação
==========

A biblioteca pode ser facilmente instalada com o Composer.

.. code-block:: sh

    $ composer require doctrine/persistence

Visão geral
===========

As interfaces e funcionalidades neste projeto evoluíram da construção de várias
implementações diferentes de mapeadores de objetos Doctrine.
A primeira implementação foi o ORM_, depois veio o `MongoDB ODM`_.
Um conjunto de interfaces comuns foi extraído de ambos os projetos e lançado no
projeto `Doctrine Common`_.
Ao longo dos anos, mais funcionalidades comuns foram extraídas e, eventualmente,
movidas para este projeto independente.
Depois disso, esse projeto foi dividido em vários projetos, incluindo este.

Um mapeador de objetos Doctrine se parece com isso quando implementado.

.. code-block:: php

    final class User
    {
        /** @var string */
        private $username;

        public function __construct(string $username)
        {
            $this->username = $username;
        }

        // ...
    }

    $objectManager = new ObjectManager();
    $userRepository = $objectManager->getRepository(User::class);

    $newUser = new User('jwage');

    $objectManager->persist($newUser);
    $objectManager->flush();

    $user = $objectManager->find(User::class, 1);

    $objectManager->remove($user);
    $objectManager->flush();

    $users = $userRepository->findAll();

Para saber mais sobre as interfaces e funcionalidades completas, continue lendo!

ObjectManager
=============

A principal interface pública que uma pessoa usuária final usará é a interface
``Doctrine\Persistence\ObjectManager``.

.. code-block:: php

    namespace Doctrine\Persistence;

    interface ObjectManager
    {
        public function find($className, $id);
        public function persist($object);
        public function remove($object);
        public function clear();
        public function detach($object);
        public function refresh($object);
        public function flush();
        public function getRepository($className);
        public function getClassMetadata($className);
        public function getMetadataFactory();
        public function initializeObject($obj);
        public function contains($object);
    }

ObjectRepository
================

O repositório de objetos é usado para recuperar instâncias dos seus objetos
mapeados do mapeador.

.. code-block:: php

    namespace Doctrine\Persistence;

    interface ObjectRepository
    {
        public function find($id);
        public function findAll();
        public function findBy(array $criteria, ?array $orderBy = null, $limit = null, $offset = null);
        public function findOneBy(array $criteria);
        public function getClassName();
    }

Mapeamento
==========

Para que o Doctrine consiga persistir seus objetos em um armazenamento de dados,
você precisa mapear as classes e propriedades de classe para que elas possam ser
armazenadas e recuperadas adequadamente, mantendo um estado consistente.

ClassMetadata
-------------

.. code-block:: php

    namespace Doctrine\Persistence\Mapping;

    interface ClassMetadata
    {
        public function getName();
        public function getIdentifier();
        public function getReflectionClass();
        public function isIdentifier($fieldName);
        public function hasField($fieldName);
        public function hasAssociation($fieldName);
        public function isSingleValuedAssociation($fieldName);
        public function isCollectionValuedAssociation($fieldName);
        public function getFieldNames();
        public function getIdentifierFieldNames();
        public function getAssociationNames();
        public function getTypeOfField($fieldName);
        public function getAssociationTargetClass($assocName);
        public function isAssociationInverseSide($assocName);
        public function getAssociationMappedByTargetField($assocName);
        public function getIdentifierValues($object);
    }

ClassMetadataFactory
--------------------

A classe ``Doctrine\Persistence\Mapping\ClassMetadataFactory`` pode ser usada
para gerenciar as instâncias de cada uma das suas classes PHP mapeadas.

.. code-block:: php

    namespace Doctrine\Persistence\Mapping;

    interface ClassMetadataFactory
    {
        public function getAllMetadata();
        public function getMetadataFor($className);
        public function hasMetadataFor($className);
        public function setMetadataFor($className, $class);
        public function isTransient($className);
    }

MappingDriver
=============

Para carregar instâncias ``ClassMetadata``, você pode usar a interface
``Doctrine\Persistence\Mapping\Driver\MappingDriver``.
Esta é a interface que faz o carregamento principal das informações de
mapeamento de onde quer que elas estejam armazenadas.
Isso pode ser em arquivos, atributos, yaml, xml, etc.

.. code-block:: php

    namespace Doctrine\Persistence\Mapping\Driver;

    use Doctrine\Persistence\Mapping\ClassMetadata;

    interface MappingDriver
    {
        public function loadMetadataForClass($className, ClassMetadata $metadata);
        public function getAllClassNames();
        public function isTransient($className);
    }

O projeto Doctrine Persistence oferece algumas implementações básicas que
facilitam a implementação de seus próprios drivers XML, YAML ou de Atributos.

FileDriver
----------

O driver de arquivos opera em um modo em que carrega os arquivos de mapeamento
de classes individuais sob demanda.
Isso requer que a pessoa usuária siga a convenção de 1 arquivo de mapeamento por
classe e os nomes dos arquivos de mapeamento devem corresponder ao nome completo
da classe, incluindo namespace, com os delimitadores de namespace '\',
substituídos por pontos '.'.

Estenda a classe ``Doctrine\Persistence\Mapping\Driver\FileDriver`` para
implementar seu próprio driver de arquivo.
Aqui está um exemplo de implementação de driver de arquivo JSON.

.. code-block:: php

    use Doctrine\Persistence\Mapping\Driver\FileDriver;

    final class JSONFileDriver extends FileDriver
    {
        public function loadMetadataForClass($className, ClassMetadata $metadata)
        {
            $mappingFileData = $this->getElement($className);

            // use o array de informações de mapeamento do arquivo para preencher a instância $metadata
        }

        protected function loadMappingFile($file)
        {
            return json_decode($file, true);
        }
    }

Agora você pode usá-lo da seguinte forma:

.. code-block:: php

    use Doctrine\Persistence\Mapping\Driver\DefaultFileLocator;

    $fileLocator = new DefaultFileLocator('/caminho/para/arquivos/de/mapeamento', 'json');

    $jsonFileDriver = new JSONFileDriver($fileLocator);

Agora, se você tem uma classe chamada ``App\Model\User``, pode carregar as
informações de mapeamento da seguinte forma:

.. code-block:: php

    use App\Model\User;
    use Doctrine\Persistence\Mapping\ClassMetadata;

    $classMetadata = new ClassMetadata();

    // procura um arquivo em /caminho/para/arquivos/de/mapeamento/App.Model.User.json
    $jsonFileDriver->loadMetadataForClass(User::class, $classMetadata);


PHPDriver
---------
O ``PHPDriver`` inclui arquivos PHP que apenas preenchem instâncias
``ClassMetadata`` com código PHP puro.

.. code-block:: php

    use Doctrine\Persistence\Mapping\Driver\PHPDriver;

    $phpDriver = new PHPDriver('/caminho/para/arquivos/de/mapeamento');

Agora você pode usá-lo da seguinte maneira:

.. code-block:: php

    use App\Model\User;
    use Doctrine\Persistence\Mapping\ClassMetadata;

    $classMetadata = new ClassMetadata();

    // procura um arquivo PHP em /caminho/para/arquivos/de/mapeamento/App.Model.User.php
    $phpDriver->loadMetadataForClass(User::class, $classMetadata);

Dentro do arquivo ``/caminho/para/arquivos/de/mapeamento/App.Model.User.php``
você pode escrever código PHP puro para popular uma instância ``ClassMetadata``.
Você terá acesso a uma variável chamada ``$metadata`` dentro do arquivo que você
pode usar para popular os metadados de mapeamento.

.. code-block:: php

    use App\Model\User;

    $metadata->name = User::class;

    // ...

StaticPHPDriver
---------------

O ``StaticPHPDriver`` chama um método estático ``loadMetadata()`` em suas
classes de modelo, onde você pode preencher manualmente a instância
``ClassMetadata``.

.. code-block:: php

    $staticPHPDriver = new StaticPHPDriver('/caminho/para/classes');

    $classMetadata = new ClassMetadata();

    // procura um arquivo PHP em /caminho/para/classes/App/Model/User.php
    $phpDriver->loadMetadataForClass(User::class, $classMetadata);

Sua classe em ``App\Model\User`` ficaria assim:

.. code-block:: php

    namespace App\Model;

    final class User
    {
        // ...

        public static function loadMetadata(ClassMetadata $metadata)
        {
            // preenche a instância $metadata
        }
    }

Reflexão
========

O Doctrine usa reflexão para definir e obter os dados dentro de seus objetos.
A ``Doctrine\Persistence\Mapping\ReflectionService`` é a interface primária
necessária para um mapeador Doctrine.

.. code-block:: php

    namespace Doctrine\Persistence\Mapping;

    interface ReflectionService
    {
        public function getParentClasses($class);
        public function getClassShortName($class);
        public function getClassNamespace($class);
        public function getClass($class);
        public function getAccessibleProperty($class, $property);
        public function hasPublicMethod($class, $method);
    }

O Doctrine fornece uma implementação desta interface na classe chamada
``Doctrine\Persistence\Mapping\RuntimeReflectionService``.

Implementações
==============

Existem várias implementações diferentes das APIs do Doctrine Persistence.

- ORM_ - O Doctrine Object Relational Mapper é um mapeador de dados para bancos
  de dados relacionais.
- `MongoDB ODM`_ - O Doctrine MongoDB ODM é um mapeador de dados para MongoDB.
- `PHPCR ODM`_ - O Doctrine PHPCR ODM é um mapeador de dados construído sobre a
  API do PHPCR.

.. _ORM: https://www.doctrine-project.org/projects/orm.html
.. _MongoDB ODM: https://www.doctrine-project.org/projects/mongodb-odm.html
.. _PHPCR ODM: https://www.doctrine-project.org/projects/phpcr-odm.html
.. _Doctrine Common: https://www.doctrine-project.org/projects/common.html
