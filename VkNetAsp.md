### Пример авторизации VkNet в ASPNet приложении.
В данном примере будут использоваться cookies. В случае успешной авторизации в cookies запишется информация вида "vk_token" "полученный токен".
Для начала работы создатим пару классов для удобства работы с библиотекой.

Первый класс нам понадобится для получения пары Логин-Пароль от пользователя:
```csharp
public class Login
    {
        [Required(ErrorMessage = "Введите Email!")]
        public string Email { get; set; }
        [Required(ErrorMessage = "Введите пароль!")]
        public string Password { get; set; }
    }
```
Второй класс будет Singleton с открытым свойством VkApi:
```csharp
 public class VkSingleton
    {
        public static VkApi Api { get; set; }

        private static VkSingleton instance;
        public static VkSingleton Instance
        {
            get
            {
                if (instance == null)
                {
                    instance = new VkSingleton();
                }
                return instance;
            }
        }
        private VkSingleton()
        {
        }
    }
```
Далее в HomeController добавим пару методов для получения cookies и работы с библиотекой.
1) Добавим в конструктор HomeController
```csharp
 public HomeController()
        {
            VkSingleton.Api = new VkApi();
        }
```
2) Создадим методы для записи и чтения Cookies
```csharp
private void SetCookies(string cookie_key, string cookie_value)
        {
            CookieOptions cookie = new CookieOptions();
            cookie.Expires = DateTime.Now.AddDays(1); Срок действия cookies - 1 день.

            Response.Cookies.Append(cookie_key, cookie_value, cookie);
        }
        
private string ReadCookies(string cookie_key)
        {
            return Request.Cookies[cookie_key];
        }
```
3) Создадим метод для получения токена:
```csharp
 private string GetToken(Login login)
        {
            VkSingleton.Api.Authorize(new ApiAuthParams
            {
                ApplicationId = 6319657,
                Login = login.Email,
                Password = login.Password,
                Settings = Settings.All
            });

            return VkSingleton.Api.Token;
        }
```
4) Создадим метод который выведет нам список друзей авторизованого пользователя:
```csharp
private List<FriendsModel> FriendList()
        {
            var friendList = new List<FriendsModel>();

            var friend = VkSingleton.Api.Friends.Get(new VkNet.Model.RequestParams.FriendsGetParams
            {
                UserId = VkSingleton.Api.UserId,
                Fields = ProfileFields.FirstName | ProfileFields.LastName
            });

            foreach (var f in friend)
            {
                friendList.Add(new FriendsModel { FirstName = $"{f.FirstName}", LastName = $"{f.LastName}", ID = f.Id });
            }


            return friendList;
        }
```
5) Первый метод Index пометим атрибутом [HttpGet]:
```csharp
[HttpGet]
        public IActionResult Index()
        {
           
            if (Request.Cookies.ContainsKey("vk_token"))
            {
                string cookieValue = ReadCookies("vk_token");
                VkSingleton.Api.Authorize(new ApiAuthParams
                {
                    AccessToken = cookieValue
                });
                var frnd = FriendList();
                return View("MainPanel",frnd);
            }else
            {
                return View();
            }
        }
```

6) Добавим второй метод Index с атрибутом [HttpPost]:
```csharp
[HttpPost]
        public IActionResult Index(Login login)
        {
            string token = GetToken(login);
            SetCookies("vk_token", token);
            var friendList = FriendList();
            return View("MainPanel",friendList);
        }
```
Следующим шагом будет добавление В View/Index.cshtml  базовой разметки для формы авторизации:
```cshtml
@model VkWebApp.Models.Login
@{
    ViewData["Title"] = "Login Page | VK";
}
        <form asp-action="Index" method="post">
            <div asp-validation-summary="All"></div>
            <input type="text" asp-for="Email" placeholder="Email" class="text_decoration" />
            <input type="password" asp-for="Password" placeholder="Password" class="text_decoration" />
            <button type="submit" class="button">Войти</button>
        </form>
```
В View/MainPanel.cshtml добавим так же разметку для вывода списка друзей:
```cshtml
@model IEnumerable<VkWebApp.Models.FriendsModel>
@{
    ViewData["Title"] = "Main Panel | VK ";
}
    <table>
        @foreach (var item in Model)
        {
        <tr>
            <th>@item.ID</th>
            <th>@item.FirstName</th>
            <th>@item.LastName</th>
        </tr>
        }
    </table>
```
Если все прошло успешно, то в результате успешной авторизации произойдет перенаправление на страницу MainPanel где будет выведен список друзей.
