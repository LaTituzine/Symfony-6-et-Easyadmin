
# Symfony 6 et le bundle Easyadmin

Comment installer et utiliser le bundle Easyadmin dans une application Symfony 6.

*Pré-requis* : 
- Avoir créé une application Symfony 6

## Les différentes étapes 

1. **Installer le bundle Easyadmin**
```
$ composer require easycorp/easyadmin-bundle
```

2. **Créer le DashboardController**
```
$ symfony console make:admin:dashboard
```
```php
// src/Controller/Admin/DashboardController.php
namespace App\Controller\Admin;

use EasyCorp\Bundle\EasyAdminBundle\Config\Dashboard;
use EasyCorp\Bundle\EasyAdminBundle\Controller\AbstractDashboardController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class DashboardController extends AbstractDashboardController
{
    #[Route('/admin')]
    public function index(): Response
    {
        return parent::index();
    }

    // ...
}
```
A l'url /admin on peut voir la page d'accueil d'easyadmin.

3. **Création des CRUD Controller**
```
$ symfony console make:admin:crud
```

4. **Configurer DashboardController**
Exemple de configuration simple :
```php
// src/Controller/Admin/DashboardController.php
namespace App\Controller\Admin;

use App\Entity\Products;
use App\Entity\Users;
use App\Entity\Carts;
use EasyCorp\Bundle\EasyAdminBundle\Config\Dashboard;
use EasyCorp\Bundle\EasyAdminBundle\Controller\AbstractDashboardController;
use EasyCorp\Bundle\EasyAdminBundle\Router\AdminUrlGenerator;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class DashboardController extends AbstractDashboardController
{
    #[Route('/admin')]
    public function index(): Response
    {
        // Le dashboard redirige vers un CrudController
        $adminUrlGenerator = $this->container->get(AdminUrlGenerator::class);
        return $this->redirect($adminUrlGenerator->setController(ProductsCrudController::class)->generateUrl());
    }

    public function configureDashboard(): Dashboard
    {
        return Dashboard::new()
            ->setTitle('Nom du site');
    }

    public function configureMenuItems(): iterable
    {
        yield MenuItem::linkToDashboard('Dashboard', 'fa fa-home');
        yield MenuItem::linkToCrud('Products', 'fa-solid fa-tag', Products::class);
        yield MenuItem::linkToCrud('Users', 'fa-solid fa-users', Users::class);
        yield MenuItem::linkToCrud('Carts', 'fa-solid fa-cart-shopping', Carts::class);
    }
}
```
Toujours penser à importer les classes utilisées.

5. **Configurer les CrudController**
Exemple :
```php
namespace App\Controller\Admin;

use App\Entity\Products;
use EasyCorp\Bundle\EasyAdminBundle\Config\Crud;
use EasyCorp\Bundle\EasyAdminBundle\Field\IdField;
use EasyCorp\Bundle\EasyAdminBundle\Field\TextField;
use EasyCorp\Bundle\EasyAdminBundle\Field\SlugField;
use EasyCorp\Bundle\EasyAdminBundle\Field\ImageField;
use EasyCorp\Bundle\EasyAdminBundle\Field\TextareaField;
use EasyCorp\Bundle\EasyAdminBundle\Field\NumberField;
use EasyCorp\Bundle\EasyAdminBundle\Controller\AbstractCrudController;

class ProductsCrudController extends AbstractCrudController
{
    public static function getEntityFqcn(): string
    {
        return Products::class;
    }

    public function configureCrud(Crud $crud): Crud
    {
        return $crud
            ->setPageTitle('index', 'Produits')
            ->setEntityLabelInSingular('Produit');
    }

    public function configureFields(string $pageName): iterable
    {
        return [
            IdField::new('id')
                ->onlyOnIndex(),
            TextField::new('name', 'Nom du produit'),
            SlugField::new('slug')
                ->setTargetFieldName('name'),
            ImageField::new('image', 'Illustration du produit')
                ->setBasePath('build\images\uploads')
                ->setUploadDir('assets\images\uploads')
                ->setUploadedFileNamePattern('[slug]-[randomhash].[extension]')
                ->setFormTypeOptions(['attr' => [
                    'accept' => 'image/jpeg, image/png, image/gif'
                    ]
                ])
                ->setHelp('fichier jpeg/png/gif'),   
            TextareaField::new('description', 'Description'),
            NumberField::new('price', 'Prix')
                ->setNumDecimals(2),
        ];
    }
}
```
Le chemin pour l'upload d'images dépend de l'emplacement de votre dossier uploads. Dans ce cas Webpack Encore est utilisé donc les images sont uploadées dans le dossier *assets\images\uploads* puis pour être affichées elles sont cherchées dans *build\images\uploads*.

## Documentation Symfony

- <https://symfony.com/bundles/EasyAdminBundle/current/index.html>
- <https://symfony.com/bundles/EasyAdminBundle/current/dashboards.html>
- <https://symfony.com/bundles/EasyAdminBundle/current/crud.html>
- <https://symfony.com/bundles/EasyAdminBundle/current/fields.html>