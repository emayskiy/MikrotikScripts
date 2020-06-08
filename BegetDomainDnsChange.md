# Обновление настроек домена, расположенного на хостинге Beget
Скрипт для микротик, выполняющий проверку внешнего IP устройства Микротик и изменение ip-адреса домена распложенного на хостинге Beget.
Для изменения используется настроек домена использутеся метод changeRecords API: https://beget.com/ru/api/dns#changeRecords
Для работы необходимо изменить учетные данные в настройках скрипта и добавить запуск в шедулер микротик. 

```
#Main script settings
#-----------------------------------------------------------------------------------
:local begetDomain "domain.ru"
:local begetUser "user"		
:local begetPassword "pass"

# Default domain records
#-----------------------------------------------------------------------------------
:local aRecord "0.0.0.0"
:local mx10Record "mx1.beget.com"
:local mx20Record "mx2.beget.com"
:local txtRecord "v=spf1+redirect=beget.com"

# Get current Domain DNS and Extern IP for A-record
#-----------------------------------------------------------------------------------
:local domainCurrentIP [:resolve "$begetDomain"]
/tool fetch url="http://ident.me" dst-path="/BEGET_MyExternIP.txt";
:local aRecord [/file get BEGET_MyExternIP.txt contents]
# Change domain settings
:if ($aRecord != $domainCurrentIP) do={
     :log info "BEGET: $begetDomain DNS IP: $domainCurrentIP"
     :log info "BEGET: Current extern IP: $aRecord"     
	 :local updateDnsUrl "https://api.beget.com/api/dns/changeRecords?login=$begetUser&passwd=$begetPassword&input_format=json&output_format=json&input_data={\"fqdn\":\"$begetDomain\",\"records\":{\"A\":[{\"priority\":10,\"value\":\"$aRecord\"}],\"MX\":[{\"priority\":10,\"value\":\"$mx10Record\"},{\"priority\":20,\"value\":\"$mx20Record\"}],\"TXT\":[{\"priority\":10,\"value\":\"$txtRecord\"}]}}";
     :log info "BEGET: $updateDnsUrl "
     /tool fetch url=$updateDnsUrl dst-path="/BEGET_ChangeIPResult.txt";
     delay 1
     :local upresult [/file get BEGET_ChangeIPResult.txt contents] 
     :local sucpos [:find $upresult "success" -1]
     :if ($sucpos>0) do={
	      :log warning "BEGET: DNS settings for $begetDomain is changed: $upresult"          
     } else={
          :log error "BEGET: Error: $upresult"
    }
} else={
    :log info "BEGET: OK"
}
```