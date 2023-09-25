# Уведомления

О ходе выполнения операций Сбер шлет уведомления на сервисную точку (см. анкета при подключении СБП) по протоколу `https` в формате `json`. **Уведомления доставляются только один раз** - если ваш сервис не ответит, то уведомление будет потеряно и придется запрашивать статус операции отдельно.

**Важно!** Уведомления не доступны в песочнице! Протестировать работу уведомлений можно только в продакшн среде.

Пример уведомления об успешной оплате:

``` json
{
    "sbpOperationParams": {
        "sbpOperationId": "A3267042237426060000020011080701",
        "sbpPayerId": "*********6600"
    },
    "authCode": "951258",
    "orderId": "fd0241e27be9401aa709e4f7aad87eb8",
    "clientName": "Иван Иванович И.",
    "partnerOrderNumber": "0000100000046",
    "rqUid": "ce6a7d8f0b45423eb024e5c4214ba46b",
    "rqTm": "2023-09-24T04:22:37Z",
    "idQR": "AD100041695S7RT69D5R9C86C1M3EV5O",
    "operationSum": 1,
    "tid": "29525577",
    "orderState": "PAID",
    "rrn": "326717086774",
    "responseCode": "0",
    "operationDateTime": "2023-09-24T07: 22: 37Z",
    "operationId": "CE7FF8DB8B4B4105B8B3A2CC400E3A6E",
    "operationType": "PAY",
    "operationCurrency": "643",
    "memberId": "00003505"
}
```

`clientName` предоставляется только если оплачивает клиент Сбера. При оплате с Тинькофф - поле пустое.

Пример уведомления о фактическом проведении возврата (отмены):

``` json
{
    "sbpOperationParams": {
        "sbpOperationId": "A32680755411910E0000020011080701",
        "sbpPayerId": "*********6600"
    },
    "authCode": "000000",
    "orderId": "18faaa9ee4ac4794a8cf6b685d681cd0",
    "clientName": "Иван Иванович И.",
    "partnerOrderNumber": "0000100000049",
    "rqUid": "dacd57767e67403e9cdae8a9ebf57e5b",
    "rqTm": "2023-09-25T07:58:41Z",
    "idQR": "AD10002J6D31UCKR93OOP59NJ8290FK0",
    "operationSum": 1,
    "tid": "29525577",
    "orderState": "PAID",
    "responseCode": "-1",
    "operationDateTime": "2023-09-25T10:58:41Z",
    "operationId": "AF41641CB3164202AF22682B60CBB077",
    "operationType": "REFUND",
    "operationCurrency": "643",
    "memberId": "00003505"
}
```

Пример класса C# для десериализации сообщейний:

``` cs
/// <summary>
/// Уведомление в ходе процесса оплаты QR
/// </summary>
public class Notify
{
    /// <summary>
    /// Блок с перечнем параметров операции СБП. Передается только для операции оплаты через СБП (добавляется в API v3.0.0)
    /// </summary>
    public NotifySBPParams sbpOperationParams { get; set; } = new();

    /// <summary>
    /// Код авторизации
    /// </summary>
    public string authCode { get; set; }

    /// <summary>
    /// ID заказа в АС ППРБ.Карты (Сбербанк)
    /// </summary>
    public string orderId { get; set; }

    /// <summary>
    /// Маскированное Имя Отчество Ф. плательщика (добавляется в API v3.0.0)
    /// </summary>
    public string clientName { get; set; }

    /// <summary>
    /// Номер заказа в CRM Клиента
    /// </summary>
    public string partnerOrderNumber { get; set; }

    /// <summary>
    /// Уникальный идентификатор запроса ("^(([0-9]|[a-f]|[A-F]){32})$")
    /// </summary>
    public string rqUid { get; set; }

    /// <summary>
    /// Дата/время формирования запроса
    /// </summary>
    public string rqTm { get; set; }

    /// <summary>
    /// IdQR устройства, на котором сформирован заказ.
    /// </summary>
    public string idQR { get; set; }

    /// <summary>
    /// Сумма операции в минимальных единицах Валюты
    /// </summary>
    public int operationSum { get; set; }

    /// <summary>
    /// Уникальный идентификатор терминала
    /// </summary>
    public string tid { get; set; }

    /// <summary>
    /// Статус заказа
    /// </summary>
    /// <remarks>["PAID", "CREATED", "REVERSED", "REFUNDED", "REVOKED", "DECLINED", "EXPIRED", "AUTHORIZED", "CONFIRMED", "ON_PAYMENT"]</remarks>
    public string orderState { get; set; }

    /// <summary>
    /// RRN операции
    /// </summary>
    public string rrn { get; set; }

    /// <summary>
    /// Код успешности авторизации. По операциям СБ - 2 символа, по операциям Банков Партнеров 6 символов.
    /// </summary>
    public string responseCode { get; set; }

    /// <summary>
    /// Описание кода ответа
    /// </summary>
    public string responseDesc { get; set; }
    /// <summary>
    /// Дата/Время совершения операции
    /// </summary>
    public string operationDateTime { get; set; }

    /// <summary>
    /// Идентификатор операции в АС Банка (ППРБ Ecom)
    /// </summary>
    public string operationId { get; set; }

    /// <summary>
    /// Тип операции.
    /// </summary>
    /// <remarks>["PAY", "REVERSE", "REFUND", "SBP_PAY_ACKNOWL", "SBP_ACK_ONUS", "SBP_STATUS_OUT", "SBP_CREDIT_IN_RQ", "SBP_REFUND_IN_RQ"]</remarks>
    public string operationType { get; set; }

    /// <summary>
    /// Валюта операции, цифровой код по ISO 4217.
    /// </summary>
    public string operationCurrency { get; set; }

    /// <summary>
    /// Идентификатор клиента
    /// </summary>
    public string memberId { get; set; }
}

/// <summary>
/// Блок с перечнем параметров операции СБП. Передается только для операции оплаты через СБП (добавляется в API v3.0.0)
/// </summary>
public class NotifySBPParams
{
    /// <summary>
    /// Идентификатор операции в СБП
    /// </summary>
    public string sbpOperationId { get; set; }

    /// <summary>
    /// Маскированный идентификатор плательщика
    /// </summary>
    public string sbpPayerId { get; set; }
}
```
