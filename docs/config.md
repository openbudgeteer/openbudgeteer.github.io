# Configuration

Settings can be defined via Docker environment variables or via `appsettings.json` file which is located in the root folder of the application.

## Database

For database configuration see [Database](install.md#database).

## App Settings

| Variable            | Description                                                                                                | Default                 |
|---------------------|------------------------------------------------------------------------------------------------------------|-------------------------|
| APPSETTINGS_CULTURE | Localization identifier to set things like Currency, Date and Number Format. Must be a BCP 47 language tag | en-US                   |
| APPSETTINGS_THEME   | Sets the [Bootswatch](https://bootswatch.com) Theme that will be used.                                     | default                 |
