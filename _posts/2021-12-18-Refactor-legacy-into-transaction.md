# Start using transaction in a non transaction legacy app

### Symfony Event VS Decorator VS Template Method 

### Template Method
This method will force us each handler to have a method `execute()` but the downside would be that we need to add the `MessageBusInterface` for each handler.
The good side is that we make sure everytime this handler is executed then it is going to be via Transaction.
```PHP

abstract class AbstractHandler {
    
    public function handle(Command $command): void {
        $this->pdo->startTransaction();
        try {
            $this->execute($command);
            $this->pdo->commit();

            $this->dispatchIntegrationEvents();
        } catch (\Exception $e) {
            $this->pdo->rollBack();
        }
    }

    abstract private function execute(Command $command): void;
}

class SomeHandler extends AbstractHandler {
    private function execute(Command $command): void;
}

```

### Symfony Event
This method is the same logic as Template Method, we need to dispatch an event at the end of the handler, this can be achieved via Decorator or TemplateMethod.

```PHP
abstract class AbstractHandler {
    
    public function handle(Command $command): void {
        $this->pdo->startTransaction();
        try {
            $this->execute($command);
            $this->pdo->commit();

            //dispatch symfony event
            $this->dispatchEvent();
        } catch (\Exception $e) {
            $this->pdo->rollBack();
        }
    }

    abstract private function execute(Command $command): void;
}

class SomeHandler extends AbstractHandler {
    private function execute(Command $command): void;
}
```

### Decorator Pattern

This is the cleanes way to implement the bus, we don't need deps in the handlers/usecases.


```PHP
class DbTransactionBusDecorator {
    
    //...deps

    public function process(Command $command): void
    {
        $this->startDbTransaction();

        try {
            $this->commandBus->process($command);
            $this->commitDbTransaction();
        } catch (\Throwable $ex) {
            $this->rollbackDbTransaction();

            throw $ex;
        }

        $this->dispatchIntegrationMessages(); // we dispatch here the integration events
    }
}
```


### Options that we have 

### How to implement it 

### What issues we will have


### 