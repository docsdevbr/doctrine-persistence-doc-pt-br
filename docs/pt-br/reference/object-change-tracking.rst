:source_url: https://github.com/doctrine/persistence/blob/4.0.x/docs/en/reference/object-change-tracking.rst
:revision: 528e2d0d1c8663a2f46b05a0bf4876a9a58b9f86
:status: ready

:title: Rastreamento de mudanças de objeto

Rastreamento de mudanças de objeto
==================================

O rastreamento de mudanças é o processo de determinar o que mudou em objetos
observados desde a última vez que eles foram sincronizados com o backend de
persistência.

Esta abordagem é baseada no
`padrão observador <https://pt.wikipedia.org/wiki/Observer>`_ e consiste nas
duas interfaces a seguir:

 * ``Doctrine\Persistence\NotifyPropertyChanged``, que é implementada pelo objeto
   cujas mudanças podem ser rastreadas,
 * ``Doctrine\Persistence\PropertyChangedListener``, que é implementada por
   assinantes que estão interessados em rastrear as mudanças.

Notificando assinantes
~~~~~~~~~~~~~~~~~~~~~~

Uma classe que deseja permitir que outros objetos assinem precisa implementar a
interface ``NotifyPropertyChanged``.
Como uma diretriz, tal implementação pode parecer com a seguinte:

.. code-block:: php

    <?php

    use Doctrine\Persistence\NotifyPropertyChanged;
    use Doctrine\Persistence\PropertyChangedListener;

    class MyTrackedObject implements NotifyPropertyChanged
    {
        // ...

        /** @var PropertyChangedListener[] */
        private $listeners = [];

        public function addPropertyChangedListener(PropertyChangedListener $listener) : void
        {
            $this->listeners[] = $listener;
        }
    }

Então, em cada mutador dessa classe ou de quaisquer classes derivadas, você
precisa notificar todas as instâncias de ``PropertyChangedListener``.
Como exemplo, adicionamos um método de conveniência em ``MyTrackedObject`` que
mostra esse comportamento:

.. code-block:: php

    <?php

    // ...

    class MyTrackedObject implements NotifyPropertyChanged
    {
        // ...

        final protected function notifySubscribers(string $propertyName, $oldValue, $newValue) : void
        {
            foreach ($this->listeners as $listener) {
                $listener->propertyChanged($this, $propertyName, $oldValue, $newValue);
            }
        }

        public function setAge(int $age) : void
        {
            if ($this->age === $age) {
                return;
            }

            $this->notifySubscribers('age', $this->age, $age);
            $this->age = $age;
        }
    }

Você tem que invocar ``notifySubscribers()`` dentro de cada método que altera o
estado persistente de ``MyTrackedObject``.

A verificação se o novo valor é diferente do antigo não é obrigatória, mas
recomendada.
Dessa forma, você também tem controle total sobre quando considera uma
propriedade alterada.
