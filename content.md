# Image Uploads Cheatsheet

The CarrierWave and Cloudinary gems provide us with an easy way to allow image
uploads through forms.

## Installation

In your `Gemfile`, include

```ruby
gem "carrierwave"
gem "cloudinary"
```

and then run `bundle install` in the terminal.

In the file `config/environment.rb`, add these require statments:

```ruby{3-4}
# Load the Rails application.
require_relative "application"
require "carrierwave"
require "carrierwave/orm/activerecord"

# Initialize the Rails application.
Rails.application.initialize!
```

## Getting started with Carrierwave

### Add a column to store the file's URL

You need to have a column of type `String` in whichever table that you want to
attach a file upload to. This column will, ultimately, store the URL of the
uploaded file after CarrierWave hosts it.

If you already included the column when you generated your resource, great. Skip
to the next step.

If not, you have to add it. For example, here we add a column called `avatar` to
the users table, which we plan to store an image in:

```
rails generate migration add_avatar_to_users avatar:string
rake db:migrate
```

In your case, it may be a song, transcript, image, etc.

### Generate an uploader

From the command line:

```
rails generate uploader Avatar
```

In your case, change `Avatar` to match whatever you are trying to upload / the
column you added to the table: Song, Transcript, Image, etc.

**Restart your `bin/dev`.**

### Mount the uploader in the model

In the relevant model add `mount_uploader :avatar, AvatarUploader`.

```ruby{4}
# app/models/user.rb

class User < ApplicationRecord
  mount_uploader :avatar, AvatarUploader
end
```

Again, customize to match your use case. `:avatar` should be the column you
created, and `AvatarUploader` should be whatever uploader you generated.

### Enhance the form

We need an `<input type="file">` in our form to handle the upload. If you
already have a `type="text"` input, change it; or if you don't, add a new one:

```erb
<input type="file" id="avatar" name="avatar" class="form-control">
```

We also need to change the `<form>` tag itself to allow files to be uploaded by
adding the `enctype="multipart/form-data"` attribute:

```erb
<form action="/create_user" method="post" class="form-horizontal" enctype="multipart/form-data">
```

## Assign the file just like any other params

In your create/update actions, assign the new file upload just like any other
form parameter:

```ruby
@user.avatar = params.fetch(:avatar)
```

That should be it!

## Retrieving the upload

CarrierWave will, by default, process the upload and then store it in your
`public` folder, so that you can use it just like any other URL on the internet.

For example, we could now do

```erb
<img src="<%= user.avatar %>" class="img-responsive">
```

## Adding Cloudinary for storage

CarrierWave works fine locally at this point, but it's going to break if you try
to host your project live on Heroku or some other platform. That's because Heroku and other services won't let you permanently write to the public folder of a live application.

If you're going to deploy your app, then you'll need to find a file
hosting service, like Amazon AWS S3, to store any uploaded files. Fortunately,
there's a great service called Cloudinary that makes it easy to host uploaded
images; and they have a generous free tier.

### Update your ImageUploader

To integrate Cloudinary, first adjust your `ImageUploader` file so it looks like:

```ruby
class ImageUploader < CarrierWave::Uploader::Base
  include Cloudinary::CarrierWave
end
```

Most importantly, make sure you remove the line that says:

```ruby
storage :file
```

### Configure Temporary Storage for CarrierWave

Create an initializer file called `carrierwave.rb` in `config/initializers` and add the following content:

```ruby
CarrierWave.configure do |config|
  config.cache_storage = :file
end
```

### Retrieve Cloudinary API info

Next, you'll need to [sign up for a Cloudinary
account](https://cloudinary.com/users/register/free) and get
your API info. You can find your cloud name by going to Settings and clicking
the Account tab. You can find your API key and secret by going to Settings and
clicking the Security tab.

### Add Cloudinary API info in Environment Variables

For security, it's best to store your keys in environment variables. Read more
on the how and why this is important in the guide on [storing your credentials
securely](https://learn.firstdraft.com/lessons/52-storing-credentials-securely).

Depending on what service you are hosting your deployed app on (if you choose to deploy it), you will need to manually add the environment variables to your production app. For instance, [here's how to do so on Render](https://render.com/docs/configure-environment-variables#configuring-secrets-and-other-environment-information-on-render).

### Store Cloudinairy API info in an initializer

Finally, create an initializer file called `cloudinary.rb` in `config/initializers` and add the following content:

```ruby
Cloudinary.config do |config|
  config.cloud_name = ENV["CLOUDINARY_CLOUD_NAME"]
  config.api_key = ENV["CLOUDINARY_API_KEY"]
  config.api_secret = ENV["CLOUDINARY_API_SECRET"]
  config.cdn_subdomain = true
end
```

## Further Reading

- [Official docs](https://github.com/carrierwaveuploader/carrierwave)
- [Railscast #253](http://railscasts.com/episodes/253-carrierwave-file-uploads)
- [Cloudinary Carrierwave integration](https://cloudinary.com/documentation/rails_carrierwave)

---
