# Итоговый проект: Настройка корпоративной локальной сети для нескольких филиалов

## Описание задания

В этом проекте необходимо организовать локальную сеть для нескольких филиалов компании с учётом указанных в таблице количества сотрудников, отделов и филиалов. Сеть будет поддерживать автоматическое назначение IP-адресов, защищённые подключения, динамическую маршрутизацию и VLAN для каждого отдела. Связь между филиалами будет обеспечена, а каждому отделу предоставлен свой VLAN с DHCP-сервером для автоматического получения IP-адресов сотрудниками.

## Состав сети и распределение ресурсов

### Общие параметры

- **Количество сотрудников**: 92
- **Количество отделов**: 4 (по 23 сотрудника в каждом отделе)
- **Количество филиалов**: 3
  - Филиалы 1 и 2 содержат отделы 1 и 2.
  - Филиал 3 содержит отдел 3.
  - Филиал 4 содержит отдел 4.
  
### Сетевая инфраструктура

- **Коммутаторы**: каждый отдел получает один или более коммутаторов, рассчитанных с учетом подключения по 3 ПК на каждый коммутатор.
- **Агрегация каналов**: коммутаторы одного отдела соединяются в последовательную цепочку, используя агрегацию LACP для увеличения пропускной способности и отказоустойчивости.
- **DHCP-сервер**: централизованный DHCP-сервер размещён в одном из филиалов и использует команду `ip helper-address` для распределения IP-адресов по VLAN.
- **Назначение VLAN**: для каждого отдела создается отдельный VLAN с наименованиями, отражающими отдел: `first_depart`, `second_depart` и т.д. Номера VLAN выбираются произвольно, за исключением VLAN 1 и 4096.

### Настройка подсетей

- **Адресация отделов**: для каждого отдела выбирается подсеть с оптимальной маской, которая минимально покрывает количество сотрудников (например, для 23 сотрудников подходит маска /27).
- **Связь между филиалами**: настроена межфилиальная подсеть с маской /30 для экономии адресного пространства и обеспечения стабильного обмена данными между филиалами.

### Защита сети

- **Защита устройств**: все устройства защищены паролями и имеют IP-адреса управления в единой сети управления.
- **Port-security**: на каждом ПК включен Port-security, допускающий одно соединение по MAC-адресу. При подключении другого устройства порт автоматически отключается.

## Шаги настройки

### 1. Подключение и настройка коммутаторов

1. **Подключите коммутаторы в последовательную цепочку с агрегацией каналов** (LACP) в каждом отделе.
2. Настройте VLAN на каждом коммутаторе согласно отделу:
   ```bash
   vlan <номер_VLAN>
   name first_depart (или имя соответствующего отдела)
   ```
3. Настройте порты коммутаторов для доступа и trunk-подключений между коммутаторами в LACP.

### 2. DHCP для каждого отдела

1. Разместите DHCP-сервер в одном из филиалов.
2. Настройте DHCP для каждого отдела, используя `ip helper-address` на коммутаторах для пересылки DHCP-запросов.
3. Настройте пул DHCP для каждого VLAN, учитывая соответствующую подсеть и маску, оптимизированную под количество сотрудников.

### 3. Настройка маршрутизации

1. Настройте динамическую маршрутизацию (например, OSPF или EIGRP) для связи между всеми филиалами.
2. При необходимости настройте статические маршруты (если фамилия не указана как требующая только динамическую маршрутизацию).
3. Убедитесь, что каждая подсеть в филиале может достигать других филиалов и отделов.

### 4. Защита сети

1. Установите пароли для доступа к каждому устройству.
2. Настройте port-security на каждом ПК, позволяя одному MAC-адресу подключаться к каждому порту:
   ```bash
   switchport port-security
   switchport port-security maximum 1
   switchport port-security violation shutdown
   ```

## Проверка выполнения задания

1. Убедитесь, что все устройства получают IP-адрес автоматически из соответствующих подсетей.
2. Выполните тесты для проверки связи между всеми филиалами с помощью команды `ping`.
