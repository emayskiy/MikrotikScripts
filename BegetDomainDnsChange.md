# Обновление настроек домена, расположенного на хостинге Beget
Скрипт для микротик, выполняющий проверку внешнего IP-адреса устройства Микротик и изменение настроек днс домена, распложенного на хостинге Beget.

Использутеся метод API changeRecords: https://beget.com/ru/api/dns#changeRecords

Для работы необходимо изменить учетные данные в настройках скрипта и добавить его в планировщик задач микротик. 

> У API есть одна особенность: нельзя изменить только один тип записи ДНС. При вызове метода в настройках ДНС домена удаляются все типы записи и добавляются только те, которые передали в параметрах. Поэтому нужно обязательно передавать все типы записей: A, MX, TXT.

```
#Main script settings
#-----------------------------------------------------------------------------------
:local begetDomain "domain.ru"
:local begetUser "user"		
:local begetPassword "pass"
:local dDNSInterfaceName "WanInterfaceName"

# Default domain records
#-----------------------------------------------------------------------------------
:local aRecord "0.0.0.0"
:local mx10Record "mx1.beget.com"
:local mx20Record "mx2.beget.com"
:local txtRecord "v=spf1+redirect=beget.com"

# Get current Domain DNS and Extern IP for A-record
#-----------------------------------------------------------------------------------
:local domainCurrentIP [:resolve "$begetDomain"]

# Получение внешнего IP исползуя сервис ident.me
#/tool fetch url="http://ident.me" dst-path="/BEGET_MyExternIP.txt";
#:local aRecord [/file get BEGET_MyExternIP.txt contents]

# Получение внешнего IP из настроек Asterisk (Если на роутере несколько провайдеров)
:local aRecord [ /ip address get [/ip address find interface=$dDNSInterfaceName ] address ] 
:local aRecord [:pick $aRecord 0 [:find $aRecord "/"]]
# Change domain settings
:if ($aRecord != $domainCurrentIP) do={
     :log info "BEGET: $begetDomain DNS IP: $domainCurrentIP"
     :log info "BEGET: Current extern IP: $aRecord"     
	 :local updateDnsUrl "https://api.beget.com/api/dns/changeRecords?login=$begetUser&passwd=$begetPassword&input_format=json&output_format=json&input_data={\"fqdn\":\"$begetDomain\",\"records\":{\"A\":[{\"priority\":10,\"value\":\"$aRecord\"}],\"MX\":[{\"priority\":10,\"value\":\"$mx10Record\"},{\"priority\":20,\"value\":\"$mx20Record\"}],\"TXT\":[{\"priority\":10,\"value\":\"$txtRecord\"}]}}";
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
