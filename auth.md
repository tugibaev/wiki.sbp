# Авторизация

Для выполнения любого запроса необходимо запросить токен. Чтобы запросить токен необходимо иметь:

1. `ClientID`
2. `SecretID`
3. Сертификат

Все можно получить в [SBER API REGISTRY](https://api.developer.sber.ru/)

Примерная функция получения токена для заданного `scope`:

``` cs
private async Task<string> CreateTokenAsync(string scope, SettingsPack settings)
{
    using var handler = new HttpClientHandler()
    {
        ClientCertificateOptions = ClientCertificateOption.Manual,
    };

    handler.ClientCertificates.Add(settings.Certificate);

    using var client = new HttpClient(handler);

    client.DefaultRequestHeaders.Add("rquid", Utils.NewID());
    client.DefaultRequestHeaders.Add("Accept", "application/json");
    client.DefaultRequestHeaders.Authorization =
        new AuthenticationHeaderValue(
            "Basic", Convert.ToBase64String(
                System.Text.Encoding.ASCII.GetBytes(
                   $"{settings.ClientID}:{settings.SecretID}")));


    var url = new Uri("https://mc.api.sberbank.ru:443/prod/tokens/v3/oauth");
    var fields = new Dictionary<string, string>
    {
        { "grant_type", "client_credentials" },
        { "scope", $"https://api.sberbank.ru/{scope}" }
    };
    var data = new FormUrlEncodedContent(fields);
    var answer = await client.PostAsync(url, data);

    while (true)
    {
        if (answer.IsSuccessStatusCode)
        {
            var answer_string = await answer.Content.ReadAsStringAsync();
            var sberanswer = JsonSerializer.Deserialize<SberAuth>(answer_string);

            return sberanswer.access_token;
        }

        if (answer.StatusCode == System.Net.HttpStatusCode.Unauthorized)
        {
            throw new Exception("Не верные данные авторизации шлюза");
        }
    }
}
```
Класс для десериализации ответа сервера:

``` cs
public class SberAuth
{
    public string access_token { get; set; }
    public int expires_in { get; set; }
    public string scope { get; set; }
    public string session_state { get; set; }
    public string token_type { get; set; }

}
```
