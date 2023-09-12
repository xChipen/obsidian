[[MVC]]

```
        [Route("SystemMenu/Get")]
        [HttpPost, ActionName("Get")]
        public HttpResponseMessage Get(SystemMenu_Query data)
        {
            Log.Info("API : SystemMenu/Get");

            SystemMenu_Get_rs rs = SystemMenuDAO.systemMenu_Get();

            return CommDAO.getResponse(Request, rs);
        }
```