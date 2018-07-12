### Пример авторизации VkNet в ASPNet приложении.
В данном примере будут использоваться cookies. В случае успешной авторизации в cookies запишется информация вида "vk_token" "полученный токен".
Для начала создадим класс для работы с VkNet где будет содержаться метод для авторизации и метод для получения списка друзей авторизованного пользователя.
```csharp
public class VkModel
    {
        private VkApi vkApi = new VkApi();


        public string Authorize(Login login)
        {
            vkApi.Authorize(new ApiAuthParams
            {
                ApplicationId = ,// ID вашего приложения.
                Login = , // Логин.
                Password =, // Пароль.
                Settings = Settings.All
            });

            return vkApi.Token;
        }

        public List<FriendsModel> GetFriends()
        {
            var friendlist = new List<FriendsModel>();

            var friends = vkApi.Friends.Get(new FriendsGetParams
            {
                UserId = vkApi.UserId,
                Fields = ProfileFields.FirstName | ProfileFields.LastName

            });

            foreach (var friend in friends)
            {
                friendlist.Add(new FriendsModel { FirstName = $"{friend.FirstName}", LastName = $"{friend.LastName}", ID = friend.Id });
            }
            return friendlist;
        }

        
    }
}
```
