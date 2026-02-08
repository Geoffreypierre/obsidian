<h1 style="width: 260px; background: linear-gradient(135deg, #1a1a2e 0%, #16213e 40%, #0f3460 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; font-size: 42px; font-weight: 700; margin: 35px auto; text-align: center; filter: drop-shadow(0 0 8px rgba(99,102,241,0.3))">Symfony</h1>

<div id="toc" style="margin: 0 auto 40px auto; max-width: 380px; padding: 20px 28px; border-radius: 10px; background: #1e1e2e; border: 1px solid #313244;">

<h3 style="margin: 0 0 14px 0; font-size: 15px; font-weight: 600; color: #cdd6f4; letter-spacing: 0.08em; text-transform: uppercase;">— Table des matières</h3>

<ul style="margin: 0; padding: 0; list-style: none; display: flex; flex-direction: column; gap: 6px;">
  <li><a href="#commandes-principales" style="color: #f43f5e; text-decoration: none; font-size: 14px; font-weight: 500;">① Commandes principales</a></li>
  <li><a href="#base-de-donnees" style="color: #8b5cf6; text-decoration: none; font-size: 14px; font-weight: 500;">② Base de données</a></li>
  <li><a href="#formulaires" style="color: #10b981; text-decoration: none; font-size: 14px; font-weight: 500;">③ Formulaires</a></li>
  <li><a href="#controller" style="color: #f59e0b; text-decoration: none; font-size: 14px; font-weight: 500;">④ Controller</a>
    <ul style="margin: 4px 0 0 18px; padding: 0; list-style: none; display: flex; flex-direction: column; gap: 3px;">
      <li><a href="#controller-twig" style="color: #f59e0b; text-decoration: none; font-size: 13px; opacity: 0.75;">└ Twig</a></li>
    </ul>
  </li>
  <li><a href="#authentification" style="color: #06b6d4; text-decoration: none; font-size: 14px; font-weight: 500;">⑤ Authentification</a>
    <ul style="margin: 4px 0 0 18px; padding: 0; list-style: none; display: flex; flex-direction: column; gap: 3px;">
      <li><a href="#inscription" style="color: #06b6d4; text-decoration: none; font-size: 13px; opacity: 0.75;">└ Inscription</a></li>
      <li><a href="#inscription-formulaire" style="color: #06b6d4; text-decoration: none; font-size: 13px; opacity: 0.75;">└ Formulaire</a></li>
    </ul>
  </li>
  <li><a href="#connexion" style="color: #8b5cf6; text-decoration: none; font-size: 14px; font-weight: 500;">⑥ Connexion</a>
    <ul style="margin: 4px 0 0 18px; padding: 0; list-style: none; display: flex; flex-direction: column; gap: 3px;">
      <li><a href="#connexion-security-yaml" style="color: #8b5cf6; text-decoration: none; font-size: 13px; opacity: 0.75;">└ Security.yaml</a></li>
      <li><a href="#connexion-controller" style="color: #8b5cf6; text-decoration: none; font-size: 13px; opacity: 0.75;">└ Controller</a></li>
      <li><a href="#connexion-twig" style="color: #8b5cf6; text-decoration: none; font-size: 13px; opacity: 0.75;">└ Twig</a></li>
    </ul>
  </li>
</ul>

</div>

<h2 id="commandes-principales" style="margin-right: 100%; width: 300px; background: linear-gradient(135deg, #e11d48 0%, #f43f5e 50%, #fb7185 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; font-size: 24px; font-weight: 600;">Commandes principales</h2>

```bash
php bin/console make:controller NameController
php bin/console make:entity NameEntity
php bin/console make:user
php bin/console make:auth
```

<h2 id="base-de-donnees" style="margin-right: 100%; width: 250px; background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 50%, #a78bfa 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; font-size: 24px; font-weight: 600;">Base de données</h2>

```bash
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

```
DATABASE_URL="sgbd://username:<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="becedfcdcdc9d1ccdafe8f8c89908e908e908f">[email&#160;protected]</a>:3306/databasename"
ex:
DATABASE_URL="mysql://root:@127.0.0.1:3306/forumBD"
```

<h2 id="formulaires" style="margin-right: 100%; width: fit-content; background: linear-gradient(135deg, #059669 0%, #10b981 50%, #34d399 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; font-size: 24px; font-weight: 600;">Formulaires</h2>

```bash
php bin/console make:form FormType Entity
```

<h2 id="controller" style="margin-right: 100%; width: fit-content; background: linear-gradient(135deg, #d97706 0%, #f59e0b 50%, #fbbf24 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; font-size: 24px; font-weight: 600;">Controller</h2>

```php
$entity = new Entity();
$form = $this->createForm(EntityType::class, $entity,
    [
        'method' => 'POST',
        # URL vers lequel le formulaire sera envoyé
        'action' => $this->generateURL('route')
    ]);

$form->handleRequest($request);

if ($form->isSubmitted() && $form->isValid()) {
    $entity->setAttribute($this->getAttribute());
    # Ex : $publication->setAuteur($this->getUser());
    $entityManager->persist($entity);
    $entityManager->flush();
    $this->addFlash('success', 'Message de succès');
    return $this->redirectToRoute('route');
} else {
    $this->addFlash('error', 'Message d\'erreur');
}

return $this->render("file.html.twig",
    [
        "attribute" => $attribute,
        "form" => $form
    ]);
```

<h3 id="controller-twig" style="margin-right: 100%; width: fit-content; background: linear-gradient(135deg, #d97706 0%, #f59e0b 50%, #fbbf24 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; font-size: 20px; font-weight: 600;">Twig</h3>

```twig
{{ form_start(form) }}
<fieldset>
    <legend>Formulaire</legend>
    <div>
        {# textarea généré, avec le placeholder "Exemple de placeholder" #}
        {{ form_widget(form.field, {"attr" : {"placeholder": "Exemple de placeholder"}}) }}
    </div>
    <div>
        {# Input avec un identifiant ajouté #}
        {{ form_widget(form.field, {'id': "identifiant"}) }}
    </div>
</fieldset>
{{ form_rest(form) }}
{{ form_end(form) }}
```

<h2 id="authentification" style="margin-right: 100%; width: fit-content; background: linear-gradient(135deg, #0891b2 0%, #06b6d4 50%, #22d3ee 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; font-size: 24px; font-weight: 600;">Authentification</h2>

<h3 id="inscription" style="margin-right: 100%; width: fit-content; background: linear-gradient(135deg, #0891b2 0%, #06b6d4 50%, #22d3ee 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; font-size: 20px; font-weight: 600;">Inscription</h3>

```php
#[Route('/inscription', name: 'inscription', methods: ["GET", "POST"])]
public function inscription(Request $request,
                            EntityManagerInterface $entityManager,
                            UtilisateurRepository $utilisateurRepository,
                            UtilisateurManagerInterface $utilisateurManager,
): Response
{
    if ($this->isGranted('ROLE_USER')) {
        return $this->redirectToRoute('feed');
    }

    $utilisateur = new Utilisateur();
    $form = $this->createForm(UtilisateurType::class, $utilisateur,
        [
            'method' => 'POST',
            'action' => $this->generateURL('inscription')
        ]);
    $form->handleRequest($request);

    if ($form->isSubmitted() && $form->isValid()) {
        $utilisateur->setPlainPassword('plainPassword');
        $utilisateur->setProfilePhoto('fichierPhotoProfil');

        $entityManager->persist($utilisateur);
        $entityManager->flush();
        $this->addFlash('success', 'L\'utilisateur a été enregistrée avec succès.');
        return $this->redirectToRoute('home');
    }

    return $this->render("utilisateur/inscription.html.twig",
        [
            "form" => $form
        ]);
}
```

<h3 id="inscription-formulaire" style="margin-right: 100%; width: fit-content; background: linear-gradient(135deg, #0891b2 0%, #06b6d4 50%, #22d3ee 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; font-size: 20px; font-weight: 600;">Formulaire</h3>

```php
$builder
    ->add('login', TextType::class)
    ->add('adresseEmail', EmailType::class)
    ->add('plainPassword', PasswordType::class, [
        "mapped" => false,
        "constraints" => [
            new NotBlank(),
            new NotNull(),
            new Length(
                min: 8,
                max: 20
            ),
            new Regex(
                pattern: '#^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)[a-zA-Z\\d]{8,30}$#',
                message: 'Mot de passe invalide'
            )
        ]
    ])
    ->add('fichierPhotoProfil', FileType::class, [
        "required" => false,
        "mapped" => false,
        "constraints" => [
            new File(
                maxSize: '10M',
                maxSizeMessage: 'Fichier trop gros',
                extensions: ['jpg', 'png'],
                extensionsMessage: 'Fichier format invalide',
            )
        ],
        "attr" => [
            "accept" => 'image/png, image/jpeg',
        ]
    ])
    ->add('inscription', SubmitType::class);
```

<h2 id="connexion" style="margin-right: 100%; width: fit-content; background: linear-gradient(135deg, #7c3aed 0%, #8b5cf6 50%, #a78bfa 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; font-size: 24px; font-weight: 600;">Connexion</h2>

<h3 id="connexion-security-yaml" style="margin-right: 100%; width: fit-content; background: linear-gradient(135deg, #7c3aed 0%, #8b5cf6 50%, #a78bfa 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; font-size: 20px; font-weight: 600;">Security.yaml</h3>

```yaml
form_login:
    login_path: connection_route
    check_path: connection_route
    default_target_path: default_route
    always_use_default_target_path: true
    enable_csrf: true
logout:
    path: deconnection_route
    target: default_redirect_route
```

<h3 id="connexion-controller" style="margin-right: 100%; width: fit-content; background: linear-gradient(135deg, #7c3aed 0%, #8b5cf6 50%, #a78bfa 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; font-size: 20px; font-weight: 600;">Controller</h3>

```php
#[Route('/connexion', name: 'connexion', methods: ['GET', 'POST'])]
public function connexion(AuthenticationUtils $authenticationUtils): Response
{
    if ($this->isGranted('ROLE_USER')) {
        return $this->redirectToRoute('feed');
    }

    $lastUsername = $authenticationUtils->getLastUsername();
    return $this->render('utilisateur/connexion.html.twig', [
        'lastUsername' => $lastUsername,
    ]);
}

#[Route('/deconnexion', name: 'deconnexion', methods: ['POST'])]
public function deconnexion(): never
{
    // Ne sera jamais appelée
    throw new \Exception("Cette route n'est pas censée être appelée");
}
```

<h3 id="connexion-twig" style="margin-right: 100%; width: fit-content; background: linear-gradient(135deg, #7c3aed 0%, #8b5cf6 50%, #a78bfa 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; font-size: 20px; font-weight: 600;">Twig</h3>

```twig
<form action="{{ path('connexion') }}" method="post" class="basic-form center">
    <fieldset>
        <legend>Connexion</legend>
        <div class="access-container">
            <label for="login">Login</label>
            <input id="login" type="text" name="_username" value="{{ lastUsername }}" required/>
        </div>
        <div class="access-container">
            <label for="password">Mot de passe</label>
            <input id="password" type="password" name="_password" required/>
        </div>
        <button type="submit" class="basic-form-submit">Se connecter</b