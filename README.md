# Simple PHP Doc Parser

Simple service integration of phpstan/phpdoc-parser, with few extra goodies for practical use

## 1. Install

```bash
composer require symplify/simple-php-doc-parser
```

## 2. Register Config

Register config in your `config/config.php`:

```php
use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;
use Symplify\SimplePhpDocParser\ValueObject\SimplePhpDocParserConfig;

return static function (ContainerConfigurator $containerConfigurator): void {
    $containerConfigurator->import(SimplePhpDocParserConfig::FILE_PATH);
};
```

## 3. Usage of `SimplePhpDocParser`

Required services `Symplify\SimplePhpDocParser\SimplePhpDocParser` in constructor, where you need it, and use it:

```php
use PHPStan\PhpDocParser\Ast\PhpDoc\ParamTagValueNode;
use PHPStan\PhpDocParser\Ast\PhpDoc\PhpDocNode;
use PHPStan\PhpDocParser\Ast\Type\TypeNode;
use Symplify\SimplePhpDocParser\SimplePhpDocParser;

final class SomeClass
{
    /**
     * @var SimplePhpDocParser
     */
    private $simplePhpDocParser;

    public function __construct(SimplePhpDocParser $simplePhpDocParser)
    {
        $this->simplePhpDocParser = $simplePhpDocParser;
    }

    public function some(): void
    {
        $docBlock = '/** @param int $name */';

        /** @var PhpDocNode $phpDocNode */
        $simplePhpDocNode = $this->simplePhpDocParser->parseDocBlock($docBlock);

        // param extras

        /** @var TypeNode $nameParamType */
        $nameParamType = $simplePhpDocNode->getParamType('name');

        /** @var ParamTagValueNode $nameParamTagValueNode */
        $nameParamTagValueNode = $simplePhpDocNode->getParam('name');
    }
}
```

## 4. Traverse Nodes with `PhpDocNodeTraverser`

```php
use PHPStan\PhpDocParser\Ast\Node;
use PHPStan\PhpDocParser\Ast\PhpDoc\PhpDocNode;
use PHPStan\PhpDocParser\Ast\PhpDoc\PhpDocTagNode;
use PHPStan\PhpDocParser\Ast\PhpDoc\VarTagValueNode;
use PHPStan\PhpDocParser\Ast\Type\IdentifierTypeNode;
use Symplify\SimplePhpDocParser\PhpDocNodeTraverser;
use Symplify\SimplePhpDocParser\PhpDocNodeVisitor\AbstractPhpDocNodeVisitor;
use Symplify\SimplePhpDocParser\PhpDocNodeVisitor\CallablePhpDocNodeVisitor;

$phpDocNodeTraverser = new PhpDocNodeTraverser();
$phpDocNode = new PhpDocNode([new PhpDocTagNode('@var', new VarTagValueNode(new IdentifierTypeNode('string')))]);

// A. you can use callable to traverse
$callable = function (Node $node): Node {
    if (! $node instanceof VarTagValueNode) {
        return $node;
    }

    $node->type = new IdentifierTypeNode('int');
    return $node;
};

$callablePhpDocNodeVisitor = new CallablePhpDocNodeVisitor($callable, null);
$phpDocNodeTraverser->addPhpDocNodeVisitor($callablePhpDocNodeVisitor);

// B. or class that extends AbstractPhpDocNodeVisitor
final class IntegerPhpDocNodeVisitor extends AbstractPhpDocNodeVisitor
{
    /**
     * @return Node|int|null
     */
    public function enterNode(Node $node)
    {
        if (! $node instanceof VarTagValueNode) {
            return $node;
        }

        $node->type = new IdentifierTypeNode('int');
        return $node;
    }
}

$integerPhpDocNodeVisitor = new IntegerPhpDocNodeVisitor();
$phpDocNodeTraverser->addPhpDocNodeVisitor($integerPhpDocNodeVisitor);

// then traverse the main node
$phpDocNodeTraverser->traverse($phpDocNode);
```
