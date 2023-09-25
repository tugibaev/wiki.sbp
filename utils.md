# Сервисные функции

При работе с API Сбера необходимы некторые сервисные функции, которые многократно используются при запросах.

## NewID() - идентификатор запроса

В документации прямо рекомендуется использовать глобальный идентификатор для идентификации запросов. Я использую "синхронизированную" версию геренерации `Guid`:

``` cs
private static readonly object _lock = new();

/// <summary>
/// Генерация идентификатора запроса
/// </summary>
/// <returns></returns>
internal static string NewID()
{
    var result = string.Empty;
    lock (_lock)
    {
        result = Guid.NewGuid().ToString().Replace("-", "").ToLower();
    }
    return result;
}
```

## DateToStr(DateTime) - преобразование даты в строку

Для обращения к сервису используется специфичный формат даты с указанием временной зоны, но без ее указания (как бы это странно не звучало).

``` cs
/// <summary>
/// Преобразование даты в формат Сбера
/// </summary>
/// <returns></returns>
internal static string DateToStr(DateTime dt)
{
    return $"{dt.ToUniversalTime():s}Z";
}
```

## StrToDate(string) - преобразование строки в дату

Обратная от `DateToStr` операция.

``` cs
/// <summary>
/// Преобразование строки в дату
/// </summary>
/// <param name="date">Строка даты</param>
/// <returns></returns>
internal static DateTime? StrToDate(string date)
{
    if (DateTime.TryParse(date.TrimEnd('Z'), out var udate))
    {
        return udate.ToLocalTime();
    }

    return default;
}
```
