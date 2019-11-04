# README
les gem à rajouter dans un novueau fichier: gem 'faker', gem 'table_print', gem 'bcrypt'
créer une base de donnée postgresql
#rails new -d postgresql the_action_mailer
créer un model User avec name et email en string
#rails g model User name:string email:string
créer une BDD:
#rails db:create
migration effective
#rails db:migrate
création d'un mailer:
#rails g mailer UserMailer
ce fichier ce retrouve dans _app/mailer_

pour créer une mail:
je créer dans ma view un fichier _welcome_email.html.erb_ et un _welcome_email.text.erb_

pour envoie un envoie automatique aprés la création je rajoute dans mon model User:
#after_create :welcome_send

#  def welcome_send
#    UserMailer.welcome_email(self).deliver_now
#  end

On a utilisé un callback qui permet juste après l'inscription en base d'un nouvel utilisateur, d'appeler la méthode d'instance welcome_send. Celle-ci ne fait qu'appeler le mailer UserMailer en lui faisant exécuter welcome_email avec, pour seule entrée, l'instance créée (d'où le self).

À noter qu'on rajoute ensuite un deliver_now pour envoyer immédiatement l’e-mail. Il est possible d'utiliser un deliver_later mais son fonctionnement en production est moins évident : il faut savoir gérer les tâches asynchrones avec Active Job… On ne va pas rentrer là-dedans pour le moment.

Ici le cahier des charges serait le suivant : on veut pouvoir

vérifier que notre app Rails déclenche bien des envois d’e-mails (=> ça confirmerait que la chaîne entière d’Action Mailer est bien codée et sans bug) ;
vérifier la tronche des e-mails qu'on envoie ;
ne surtout pas envoyer des e-mails par erreur, histoire de ne pas prendre le risque de spammer de vrais clients pendant nos tests.
Pour ça on va utiliser une gem assez cool qui s'appelle _Letter Opener_. Son fonctionnement ? Dès qu'un e-mail doit être envoyé par ton app Rails, celui-ci est automatiquement ouvert dans ton navigateur web.
Testons-la immédiatement sur ton app test_action_mailer :

Mets letter_opener dans le groupe de développement de ton Gemfile puis bundle install
Maintenant va dans _config/environments/development.rb_ (fichier contenant les paramètres de ton environnement de développement) et colle les lignes
#config.action_mailer.delivery_method = :letter_opener et config.action_mailer.perform_deliveries = true
Note importante : la ligne avec perform_deliveries = true permet d'éteindre (en la passant à false) tout envoi d'email de la part de ton app Rails. C'est bon de savoir qu'elle existe !

Maintenant que la gem est installée et configurée, va dans la console Rails et créé un utilisateur à la volée (par exemple : User.create(name:"Féfé", email: "féfé@yopmail.com")). Tu devrais voir un visuel de l’e-mail que tu as rédigé en HTML s'afficher dans ton navigateur ! Si ce n'est pas le cas, tu as raté une étape de mon pas à pas…

b) Sauver la clef d'API de façon sécurisée
Une fois cette clef en main, il faut la mettre en sécurité dans ton app Rails. Pour ça, rien de mieux que la gem dotenv appliquée à Rails ! Voici les étapes :

Crée un fichier .env à la racine de ton application.
Ouvre-le et écris dedans les informations suivantes : SENDGRID_LOGIN='apikey' et SENDGRID_PWD='ta_clef_API' en remplaçant bien sûr ta_clef_API par la clef que tu viens de générer. Elle est au format SG.sXPeH0BMT6qwwwQ23W_ag.wyhNkzoQhNuGIwMrtaizQGYAbKN6vea99wc8. N'oublie pas les guillemets !
Rajoute gem 'dotenv-rails' à ton Gemfile et fait le $ bundle install
Et l'étape cruciale qu'on oublie trop souvent : ouvre le fichier .gitignore à la racine de ton app Rails et écris .env dedans.
c) Paramétrer le SMTP avec les infos de SendGrid
Parfait : tu as une clef API de SendGrid et tu es prêt à l'utiliser. Il ne te reste qu'à entrer les configurations SMTP de SendGrid dans ton app. Va dans /config/environment.rb et rajoute les lignes suivantes :

_ActionMailer::Base.smtp_settings = {
  :user_name => ENV['SENDGRID_LOGIN'],
  :password => ENV['SENDGRID_PWD'],
  :domain => 'monsite.fr',
  :address => 'smtp.sendgrid.net',
  :port => 587,
  :authentication => :plain,
  :enable_starttls_auto => true
}_


#HEROKU
Je crée une app héroku avec
#heroku create nom
ensuite retrouve cette application sur le site heroku
#reli ton api sendgrid à heroku,
va dans les parametre de ton application sur heroku
