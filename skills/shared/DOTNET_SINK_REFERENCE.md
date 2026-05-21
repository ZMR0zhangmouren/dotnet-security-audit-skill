# .NET Sink 参考（供 pipeline 与各子 Skill 统一对齐）

## SQL / ORM

- SqlCommand.ExecuteReader / ExecuteScalar / ExecuteNonQuery
- FromSqlRaw / ExecuteSqlRaw / Database.SqlQueryRaw
- Dapper Query / Execute 与动态 SQL 拼接

## NoSQL

- MongoDB Driver 的 FilterDefinition 构造
- BsonDocument 直接拼接查询条件
- Cosmos DB 自定义查询字符串

## CMD

- Process.Start
- cmd / powershell 参数拼接
- RuntimeInformation 条件分支下的 shell 调用

## SSRF

- HttpClient.SendAsync / GetAsync
- HttpWebRequest / WebRequest.Create
- GraphQL / Webhook / URL 回调客户端

## XSS / 输出

- Razor Raw / HtmlString
- Blazor MarkupString
- Controller Content / Json / 手工拼接 HTML / JS

## REDIR / 跳转

- Redirect / LocalRedirect / RedirectToAction / Results.Redirect
- Challenge / SignOut / 外部登录回跳地址

## CRLF / 响应头

- Response.Headers.Append / Add
- Set-Cookie / Location / Content-Disposition
- 下载文件名、代理透传头、自定义安全头

## FILE / WRITE / UPLOAD

- File.ReadAllText / FileStream / PhysicalFile / VirtualFileResult
- File.WriteAllText / SaveAs / IFormFile.CopyTo / CopyToAsync
- ZipArchive.ExtractToDirectory

## XXE / XML

- XmlDocument.LoadXml / XPathDocument / XDocument / XmlReader
- DtdProcessing、XmlResolver、XslCompiledTransform

## DESER

- BinaryFormatter
- NetDataContractSerializer
- Newtonsoft.Json TypeNameHandling
- LosFormatter / ObjectStateFormatter（遗留栈）

## TPL / EXPR

- Razor Runtime Compilation
- Roslyn CSharpScript
- Dynamic LINQ
- DataTable.Compute

## LDAP

- DirectorySearcher.Filter
- PrincipalContext / custom LDAP filter 字符串拼接

## AUTH / SESS / CSRF / CFG

- [Authorize] / policy / role / claim 检查
- Antiforgery token 验证
- Cookie flags / JWT validation parameters
- appsettings.json / web.config / middleware order / CORS / debug 开关
