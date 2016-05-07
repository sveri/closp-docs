{:title "Internationalization"
 :layout :page
 :page-index 1000}

In this chapter I will explain how to use internationalization with closp. We use <https://github.com/ptaoussanis/tower>
for this and everything is set up so far.


## Defining translations.

Open `src/clj/foo/example/components/locale.clj`. You will find a map similar looking to this:

    {:fallback-locale :en
         :dictionary      {:en
                           {:generic
                            {:some_error        "Some error occured."
                             :deletion_canceled "Deletion canceled."}

                            :user
                            {:email_invalid           "A valid email is required."
                             :pass_min_length         "Password must be at least 5 characters."

                            :admin
                            {:title "User Overview"}}}}

The `:fallback-locale` key defines the language that should be used if a language is requested that you have
no translation for. In this case it is english.

The dictionary setting defines the languages that you offer translation for. First, you provide the language,
`:en` in this case and then you add the sections to which the specific translation belongs.

 After that, all that is needed is a unique key and the translation: `:user/email_invalid "Invalid email"`.

## Using translations

In your code require the `[taoensso.tower :refer [t]]` namespace.
Then everytime you want to use a translated string call the function like this:

    (t locale tconfig :age)

Where
 - **t** is the function you required in the namespace
 - **locale** is the locale used for the translation
 - **tconfig** is the localization component
 - **:user/email_invalid** is the section/key to lookup in the translation map

 In closp you can aquire the locale and tconfig by taking them from the request. So in every route definition you
 can do a lookup like this:

     (routes
         (GET "/user/accountcreated" req (some-func (:locale req) (:tconfig req))))

For an extensive example look at `src/clj/foo/example/routes/user.clj`.