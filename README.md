A gettext based framework for i18n in Java and JSF applications
===============================================================

What is gettext?
----------------

Gettext is an internationalization and localization (i18n) library which is
commonly used for writing multilingual programs. Its most popular
implementation is that of the GNU project.

I18n with gettext works by marking up translatable strings in the source code,
usually by wrapping them with a function call. The xgettext tool extracts these
strings and creates a text file listing them. This file is called the template
and its name usually ends in ".pot".

The msginit tool creates a new text file mapping the extracted strings to their
translation in a given locale, having the ".po" extension. Finally the msgfmt
tool creates an optimized representation of the translation mappings that is
then used at runtime. For most programs this is a binary file ending in ".mo",
but it is also possible to create other formats, for example a java
ResourceBundle.

There are specialized editors for editing the ".mo" files, which can remember
already translated strings and contain databases of repeatedly used words.

Why use gettext instead of property bundles?
--------------------------------------------

Translation in java is usually done using property files, which map some
artificial key to the corresponding translation. In my opinion this approach
has several drawbacks.

  * The need to invent a key for each message interrupts the programmers flow.
    Sometimes it is already pretty hard to think of meaningful variable names,
    having to think about something totally unrelated to the code, like
    translation further distracts the programmer.

  * The plural handling in gettext is superior for some languages having more
    than two [plural forms][1]. For simple cases pluralization can be done using
    `ChoiceFormat`, but languages like Polish require more complicated rules.
    Additionally, the syntax for `ChoiceFormat` is probably to complicated for
    non-developers.

  * There are specialized editors for editing gettext files, which are also
    usable by non-programmers. These editors have extensive features to support
    the translator, like remembering already translated string or containing a
    terminology database.

 [1]: http://www.gnu.org/savannah-checkouts/gnu/gettext/manual/html_node/Plural-forms.html

How to use this framework in java?
----------------------------------

The first step when doing translation is actually to determine the correct
locale and ResourceBundle to look up messages. For a desktop program this is
simple since it can just use the default locale of the jvm, for a multi user
web application its more complicated. Futhermore you usually do not want to
pass around the locale to any method that might need to output messages,
neither do you want to tightly couple the java logic to the view, for example
by calling `FacesContext.getCurrentInstance`.

To solve these problems, this framework uses a `ServiceLoader` based solution
to lookup the locale and ResourceBundle, with implementations for the default
locale and JSF. The implementation classes can be specified in two files in
`META-INF/services`, `net.jhorstmann.i18n.LocaleProviderFactory` and
`net.jhorstmann.i18n.ResourceBundleProviderFactory`. To use the JSF
implementation it is enough to depend on the `i18n-jsf` library since it
contains these files in its classpath.

Translatable string in java code should be marked up by calls to the static
functions in `net.jhorstmann.i18n.I18N`. The `trc` functions take an additional
`context` parameter that is supposed to help translate words with different
meaning when used in different contexts, `trn` takes an additional parameter
for plural handling and `trnc` uses both.

In JSF views there are two ways to markup strings. The first one is to use EL
functions from the `http://jhorstmann.net/taglib/i18n` namespace. These work
just like the java functions described above. The second and preferred way is
to use the `tr` JSF component. Examples how to use this can be found in the
`index.xhtml` of the `i18n-sample-webapp` project.

Extraction of messages is supported by a maven plugin. The `i18n:gettext` goal
extracts messages from xhtml files and java **class** files, meaning it should
be invoked in the `process-classes` phase or manually after the project is
compiled.

The `i18n:init` goal creates a new ".po" file for the locale given as
`-Dlocale=xyz`. When messages are added or changed in the code, the
`i18n:gettext` and `i18n:merge` goals should be invoked to update all ".po"
files.

The `i18n:dist` goal generates the java ResourceBundle that is used at runtime
and should be bound to the `generate-resources` phase. The other goals should
be executed manually.

    <plugin>
        <groupId>net.jhorstmann.i18n</groupId>
        <artifactId>i18n-maven-plugin</artifactId>
        <version>1.0-SNAPSHOT</version>
        <configuration>
            <targetBundle>net.jhorstmann.i18n.sample.Messages</targetBundle>
            <outputFormat>class</outputFormat>
        </configuration>
        <executions>
            <execution>
                <goals>
                    <goal>dist</goal>
                </goals>
                <phase>generate-resources</phase>
            </execution>
        </executions>
    </plugin>

A JSF application has to declare its supported locale and the ResourceBundle to
use in `faces-config.xml`:

    <application>
        <locale-config>
            <default-locale>de</default-locale>
            <supported-locale>en</supported-locale>
        </locale-config>
        <resource-bundle>
            <base-name>net.jhorstmann.i18n.sample.Messages</base-name>
            <var>i18n</var>
        </resource-bundle>
    </application>

The ResourceBundle should be named "i18n" by default, this name can be
overridden in `web.xml` using the `net.jhorstmann.i18n.ResourceBundleVar`
context parameter. Alternatively, the ResourceBundle can also be specified in
`web.xml` instead of `faces-config.xml` by setting the
`net.jhorstmann.i18n.ResourceBundle` context parameter.


