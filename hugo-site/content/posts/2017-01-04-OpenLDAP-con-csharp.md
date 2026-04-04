---
title: "OpenLDAP con CSharp"
date: 2017-01-04
tags: ["csharp"]
---

En mi empresa hemos empezado a trabajar con OpenLDAP, y esto implica cambiar todos los metodos de autenticacion de los sistemas desarrollados, a éste protocolo. 

![_config.yml](/images/2017-01-04-OpenLDAP-con-csharp.gif)

Al principio parecia dificil, pero no fue asi. Todo se hizo mas facil con la ayuda de algunos articulos de stackoverflow. 

Al final pude armar una clase **helper** que me permitiera acceder a los elementos del LDAP:

~~~csharp
public class LDAPHelper
{
    private readonly LdapConnection ldapConnection;
    private readonly string searchBaseDN;
    private readonly int pageSize;

    public LDAPHelper(
        string searchBaseDN,
        string hostName,
        int portNumber,
        AuthType authType,
        string connectionAccountName,
        string connectionAccountPassword,
        int pageSize)
    {

        var ldapDirectoryIdentifier = new LdapDirectoryIdentifier(
            hostName,
            portNumber,
            true,
            false);

        var networkCredential = new NetworkCredential(
            connectionAccountName,
            connectionAccountPassword);

        ldapConnection = new LdapConnection(
            ldapDirectoryIdentifier,
            networkCredential)
        { AuthType = authType };

        ldapConnection.SessionOptions.ProtocolVersion = 3;

        this.searchBaseDN = searchBaseDN;
        this.pageSize = pageSize;
    }

    public IEnumerable<SearchResultEntryCollection> PagedSearch(
        string searchFilter,
        string[] attributesToLoad)
    {

        var pagedResults = new List<SearchResultEntryCollection>();

        var searchRequest = new SearchRequest
                (searchBaseDN,
                    searchFilter,
                    SearchScope.Subtree,
                    attributesToLoad);


        var searchOptions = new SearchOptionsControl(SearchOption.DomainScope);
        searchRequest.Controls.Add(searchOptions);

        var pageResultRequestControl = new PageResultRequestControl(pageSize);
        searchRequest.Controls.Add(pageResultRequestControl);

        while (true)
        {
            var searchResponse = (SearchResponse)ldapConnection.SendRequest(searchRequest);
            var pageResponse = (PageResultResponseControl)searchResponse.Controls[0];

            yield return searchResponse.Entries;
            if (pageResponse.Cookie.Length == 0)
                break;

            pageResultRequestControl.Cookie = pageResponse.Cookie;
        }


    }
}
~~~

Con ésta clase, la consulta a los elementos del LDAP fue sencilla:

~~~csharp
static void Main(string[] args)
        {
            try
            {
                var baseOfSearch = "dc=integrate,dc=com,dc=bo";
                var ldapHost = "192.168.0.101";
                var ldapPort = 389;
                var connectAsDN = "cn=admin,dc=integrate,dc=com,dc=bo";
                var pageSize = 1000;
                var secureString = "CONTRASEÑA_ADMIN_LDAP";

                var openLDAPHelper = new LDAPHelper(
                    baseOfSearch,
                    ldapHost,
                    ldapPort,
                    AuthType.Basic,
                    connectAsDN,
                    secureString,
                    pageSize);

                var searchFilter = "objectclass=posixAccount";
                //var searchFilter = "uid=rvera";
                var attributesToLoad = new[] { "sn","uid","cn","userPassword" };
                var pagedSearchResults = openLDAPHelper.PagedSearch(
                    searchFilter,
                    attributesToLoad);

                foreach (var searchResultEntryCollection in pagedSearchResults)
                    foreach (SearchResultEntry searchResultEntry in searchResultEntryCollection)
                    {
                        Console.WriteLine(searchResultEntry.Attributes["uid"][0] + ": " +
                                          searchResultEntry.Attributes["cn"][0]);
                        Console.WriteLine(searchResultEntry.Attributes["userPassword"][0]);
                        Console.WriteLine(".......");
                    }
            }
            catch (Exception exp)
            {
                Console.WriteLine(exp.Message);
                Console.WriteLine(exp.StackTrace);
            }

            Console.WriteLine("Presione una tecla para terminar...");
            Console.Read();
        }
~~~

Pueden obtener el ejemplo [aqui](https://bitbucket.org/re_al_/real.test.openldap)