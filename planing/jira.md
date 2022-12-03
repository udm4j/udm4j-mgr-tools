Настройки джира
===========

Планирование
-----------
1. Фильтры 

```
Core Stream - All goals and DoDs
(project = SCP AND type = Epic AND Collaboration in (DMUlyanov.SBT) OR project = CORE AND type = Epic AND summary ~ ДОД) AND statusCategory != Done

Core Stream - All tasks
(assignee in team("SC Core FE") OR Team = "SC Core FE" OR assignee in team("SC Core BE (ex SC Core Search, Auth)") OR Team in ("SC Core BE (ex SC Core Search, Auth)") OR Team in ("SC Core Tech Services 1") OR assignee in team("SC Core Tech Services 1") OR Team in ("SC Core Tech Services 2 (ex U FT 4)") OR assignee in team("SC Core Tech Services 2 (ex U FT 4)") OR Team in ("SC Новая архитектура")) AND issuetype not in (RCA) AND project not in (MET, MAN-Management, "Projects Portfolio", "Training Space")

Core Stream - out-of-date issues v2
filter = "Core Stream - All tasks" AND statusCategory != Done AND (type not in (Bug, Incident, "Release 2.0") AND status not in (IFT, "Ready for Release", "Merge Branch", "Deployed to prod", Released) AND (fixVersion in releaseDate("before 2d") OR "Фича-фриз план" < 2d) OR type not in (Bug, Incident, "Release 2.0") AND status not in (NeedTest, "in QA", IFT, "Ready for Release", "Merge Branch", "Deployed to prod", Released) AND (fixVersion in releaseDate("before 5d") OR "Фича-фриз план" < 5d) OR type in (Bug, Incident) AND status not in (IFT, "Ready for Release", "Merge Branch", "Deployed to prod", Released) AND "Установка план" < 2d OR type in (Bug, Incident) AND status not in (NeedTest, "in QA", IFT, "Ready for Release", "Merge Branch", "Deployed to prod", Released) AND "Установка план" < 5d)
```

2. Структуры и фильтры
* Core Stream: Goals and DoDs
    - Запрос
      
        ```
        Links: Add issues linked by PartOF: parent consist of children
        Links: Add sub-tasks
        Remove Inserter/Extender Duplicates
        group by: fixVersion
        sort by: summary
        insert: filter = "Core Stream - All goals and DoDs"
        ```
      
    - Дополнения 
        * Цели (Epic в пространстве SCP) декомпозированы на ДОДы (Epic в пространстве CORE) и связаны is part of/consists of
        * Цель могут быть назначены на нескольких людей 
        * Цели планируются на следующий квартал (в версию #2023q1) за месяц до его начала на 2 месяца из 3х (3й берется как резерв) 
        * Флоу у цели: новая <-> в ревью <-> закрыта, новая <-> эскалация <-> новая, любой статус <-> отменена
        * Цель закрывается только при наличии фактуры ее выполнения
        * ДОДы декомпозированы на большие задачи (Epic в бизнес-пространствах, Story в бизнес-пространствах, Change Requests в бизнес-пространствах) и связаны is part of/consists of
        * ДОД назначен на конкретного человека
        * На ДОД написал чеклист деливери лида
        * Флоу ДОДа: новая <-> в работе <-> в ревью <-> закрыта, любой статус <-> отменена
        * Отображаемые поля: key, summary, Status, Priority, Assignee, fixVersion, Team, Sum of original estimate, DoR, DoD    

* Core Stream: Epic + Story
    
    - Запрос 
    
        ```
        Links: Add sub-tasks
        Links: Add issues linked by PartOF: parent consist of children
        Group by Fix Version/s
        Group by Team
        Sort by Key
        Sort by Assignee
        Sort by Team
        Sort by Fix Version/s
        insert: project not in ("Epic Roadmap", Метрики)  AND NOT (statusCategory = Done AND (fixVersion in releasedVersions() or fixVersion in archivedVersions()  OR fixVersion is EMPTY))  AND issuetype  in (Epic, "Change Request", Story, Task)   AND  filter = "Core Stream - All tasks"
        ```
    
    - Дополнения 
        * 

Дефекты
-----------

* Дефекты без основания допуска дефектов в прод 

    - Запрос 
    
        ```
        Filter items where attribute 'isSameVersion' is not equal to 'same'
        Group by Team
        Group by Fix Version/s
        Sort by Team
        Sort by Fix Version/s
        Insert issues: filter = "Core Stream - All tasks" and issuetype in (Bug) AND resolution = Unresolved AND "Основание допуска открытого дефекта на ПРОД" is EMPTY
        ```
      
        ```
        DueDate
        
        WITH setcolor(value, color) = """{color:$color}$value{color}""":
        with dueD = if(dueDate; dueDate; cntrlDate): //cntrlDate - "Контрольный срок"
        IF (
        DAYS_BETWEEN(TODAY(), dueD)  < 2, setcolor(format_datetime(dueD,"dd.MM.yyyy"), "red"), //Red
        DAYS_BETWEEN(TODAY(), dueD)  > 1 and DAYS_BETWEEN(TODAY(), dueD) <= 5 , setcolor(format_datetime(dueD,"dd.MM.yyyy"), "orange"), //Yellow
        DAYS_BETWEEN(TODAY(), dueD)  > 5, setcolor(format_datetime(dueD,"dd.MM.yyyy"), "green") //Green
        )
        ```
        ```
        Срок SLA/RCA
      
        with sla_exceeded = IF (statusCategory == "Done"; "";
        DAYS_BETWEEN(now(); control) <  3; "{color:red} SLA - ALARM {color}";
        DAYS_BETWEEN(now(); control) <= 5; "{color:orange} SLA - WARNING {color}";
        DAYS_BETWEEN(now(); control) >  5; "{color:green} SLA - OK {color}"):
        //  "{color:green} no need {color}"):
        
        with rca_exceeded = IF (not statusCategory == "Done"; "";
        DAYS_BETWEEN(now(); resolutionDate) <= 3; "{color:red} RCA - ALARM {color}";
        DAYS_BETWEEN(now(); resolutionDate) <= 5; "{color:orange} RCA - WARNING {color}";
        DAYS_BETWEEN(now(); resolutionDate) >  5; "{color:green} RCA - OK {color}"):
        //  "{color:green} no need {color}"):
        
        with info = IF(MATCH(sla_exceeded, "") and MATCH(rca_exceeded,""); "";
        CONCAT(sla_exceeded; rca_exceeded)):
        
        info
        ```
        ```
        Есть ТК
      
        if (query;"{panel:bgColor=green}Yes{panel}";
        if (not query and key; "{panel:bgColor=red}No{panel}"; "")) 
        
      
        query=       issue in hasTestCoverage() OR issue in impactsTestResult()  or  NOT "Тест-Кейс не требуется!" is EMPTY 
        ```
        ```
        Меры
      
        if (query;"{panel:bgColor=green}Yes{panel}";
        if (not query and key; "{panel:bgColor=red}No{panel}"; "")) 
        
        
        query=      "Предпринятые меры по недопущению возникновения подобных инцидентов" is not EMPTY AND "Корневые причины возникновения проблемы" is not EMPTY
        ```
        ```
        Linked to epic
      
        if (query;"{panel:bgColor=green}Yes{panel}";
        if (not query and key; "{panel:bgColor=red}No{panel}"; "")) 
              
        query=       issueFunction in linkedIssuesOf("issueFunction in linkedIssuesOf ('key = EDU-44188', 'consist of ')")  //EDU-44188 - эпик для агрегации задач на улучшения
        ```
        ```
        Ревью тимлида
      
        if (query;"{panel:bgColor=green}Yes{panel}";
        if (not query and key; "{panel:bgColor=red}No{panel}"; "")) 
        
     
        query=       labels is not EMPTY AND labels = cs-tl-root-cause-reviewed
        ```
        ```
        Ревью QA-lead
      
        if (query;"{panel:bgColor=green}Yes{panel}";
        if (not query and key; "{panel:bgColor=red}No{panel}"; "")) 
        
        
        query=       labels is not EMPTY AND labels = cs-qa-root-cause-reviewed
        ```
        ```
        isSameVersion
      
        with i_rc_fv  = SEARCH("rc", fixVersion):
        with i_mfe_fv = SEARCH("mfe", fixVersion):
        with i_rc_av  = SEARCH("rc", affectedVersions):
        with i_mfe_av = SEARCH("mfe", affectedVersions):
        
        with i_rc_fv  = if(i_rc_fv; i_rc_fv; 0):
        with i_mfe_fv = if(i_mfe_fv; i_mfe_fv; 0):
        with i_rc_av  = if(i_rc_av; i_rc_av; 0):
        with i_mfe_av = if(i_mfe_av; i_mfe_av; 0):
        
        with error = i_rc_fv > 0 and i_rc_av = 0 or
                    i_rc_fv = 0 and i_rc_av > 0 or
                    i_mfe_fv> 0 and i_mfe_av= 0 or
                    i_mfe_fv= 0 and i_mfe_av> 0:
      
        WITH getVersion(index, version) = if(index  = 0; version; Substring(version, 0, index -2)):
        
        with s_rc_fv  = getVersion(i_rc_fv, fixVersion):
        with s_mfe_fv = getVersion(i_mfe_fv ,fixVersion):
        with s_rc_av  = getVersion(i_rc_av , affectedVersions):
        with s_mfe_av = getVersion(i_mfe_av , affectedVersions):
        
        if(
            error; "affected and fix versions are not same";
            if (fixVersion and i_rc_fv > 0 and s_rc_fv = s_rc_av; "same";
                if (fixVersion and i_mfe_fv > 0 and s_mfe_fv = s_mfe_av; "same";
                    if (fixVersion and i_rc_fv = 0 and i_mfe_fv = 0 and s_rc_fv = s_rc_av; "same";
                        if (fixVersion;"not same"; "")
                    )
                )
            )
        )
        ```
        ```
        Снижение приоритета
      
        if (JQL {priority changed};
            WITH MaxPriority = If(type != undefined and JQL{priority was Blocker};"Blocker";
                                    type != undefined and JQL{priority was Critical};"Critical";
                                    type != undefined and JQL{priority was High};"High";
                                    type != undefined and JQL{priority was Medium};"Medium";
                                    type != undefined and JQL{priority was Low};"Low"
                                ) :
            if (MaxPriority != Priority;
                CONCAT(MaxPriority," ","⇢"," ",priority)
            )
        )
        ```      
    
    - Дополнительно 
        * Отображаемые поля: key, Priority, Summary, Count, Status, Assignee, AffectedVersion, DueDate(custom),Контроль срок,Срок SLA/RCA (custom),Есть ТК(custom), Меры (custom), Linked to Epic(custom),Ревью тимлида(custom),Ревью QA(custom), isSameVersion
    
* Дефекты без импакт анализа 

    - Запрос 
        Только переданные в тестирование, исправление которых не в rc1, так как rc1 это исправление в фича-ветке - не интересно
        ```
        Group by Team
        Insert issues: filter = "Core Stream - All tasks" and project not in (SM) and type in (Bug, Incident) and status not in (Cancelled) and status was in ("in QA")  and Impact-анализ is EMPTY and fixVersion in (unreleasedVersions())  and not fixVersion  in versionMatch("r/.*rc1")
        ```

* Дефекты без обоснования просрочки SLA

    - Запрос

        ```
        Group by Team
        Insert issues: filter = "Core Stream - All tasks" AND created > 2022-09-25 AND issueFunction in linkedIssuesOf("project = SM and slaFunction = isBreached()", " -> Причина в") and not (labels is not EMPTY  and labels in (cs-itl-sla-reviewed)) and status not in (Cancelled)
        ```

* Дефекты без обоснования понижения приоритета

    - Запрос

        ```
        Filter: "Снижение приоритета" is not EMPTY
        Group by Team
        Insert issues: filter = "Core Stream - All tasks" AND type in (Bug, Incident) AND fixVersion not in (releasedVersions()) AND priority changed and not (labels is not EMPTY and labels in (cs-itl-priority-change-reviewed)) and status not in (Cancelled)
        ```

* Дефекты с некорректным приоритетом

    - Запрос

        ```
        Filter: isSameVersion is equal to 'not same'
        Group by Fix Version/s
        Group by Team
        Insert issues: filter = "Core Stream - All tasks"  and type in (Bug, Incident) and "Основание допуска открытого дефекта на ПРОД" is not EMPTY  and fixVersion in (unreleasedVersions()) and priority not in (Low, Lowest) and status not in  (Cancelled, IFT, "Merge Branch", Resolved) and project not in (CORE)
        ```
    
* Дефекты открытые

    - Запрос 
    
        ```
        Group by Team
        Group by Fix Version/s
        Sort by Team
        Sort by Fix Version/s
        Sort by Priority
        Insert issues: type in (Bug, Incident) and status not in (Cancelled) AND filter = "Core Stream - All tasks" AND NOT (statusCategory = Done AND (fixVersion in (releasedVersions(), archivedVersions()))) and not (project ="Service Management" and type in (Incident) and issueFunction in linkedIssuesOf('project not in ("Service Management") and type in (Incident)', '<- Следствие'))
        ```
      
    - Дополнительно 
    
* Риски 

    - Запрос 
    
        ```
        Insert issues: type in (Риск) and statusCategory != Done and filter = "Core Stream - All tasks"
        ```
      
    - Дополнительно 
    
* Дефекты для RCA 

    - Запрос 
    
        ```
        Group by Team
        Sort by Team
        Sort by Fix Version/s
        Sort by Priority
        insert issues: ( filter = "Core Stream - All tasks"  or  issueFunction in linkedIssuesOf("project = SM  and Аллокация in ('SC Core BE (ex SC Core Search, Auth)', 'SC Core FE', 'SC Core Tech Services 1', 'SC Core Tech Services 2 (ex U FT 4)') "))  AND project not in ("Service Management") AND status not in (Cancelled, Open) AND (type in (Incident) or type in (Bug) and  "Этап обнаружения" = ИФТ )  AND NOT (labels is not EMPTY AND labels = Автотесты) AND (  NOT (issue in hasTestCoverage() OR issue in impactsTestResult()  or  NOT "Тест-Кейс не требуется!" is EMPTY ) OR NOT ("Предпринятые меры по недопущению возникновения подобных инцидентов" is not EMPTY AND "Корневые причины возникновения проблемы" is not EMPTY)  or  NOT issueFunction in linkedIssuesOf("issueFunction in linkedIssuesOf ('key = EDU-44188', 'consist of ')")    or not ( labels is not EMPTY AND labels = cs-qa-root-cause-reviewed)) AND created >= 2021-10-01 AND created < -2d  ORDER BY cf[10201] ASC, cf[11402] ASC
        ```
    - Дополнительно 
    
* Дефекты без ревью ITL 

    - Запрос 
    
        ```
        Insert issues: filter = "Core Stream - All tasks" AND project not in ("Service Management") AND (type in (Incident) OR type in (Bug) AND "Этап обнаружения" = ИФТ) AND labels not in (cs-itl-root-cause-reviewed, Автотесты) AND labels = cs-qa-root-cause-reviewed AND created >= 2022-07-01 ORDER BY created ASC
        ```


Проблемы ведения джиры
-----------

* Легенда 
    - Легенда: флаги — для тимлидов, блокнот - для Ульянова 
    - Легенда: красное — грубое нарушение, желтое - среднее, серое - драфт

(далее - критичные нарушения)

1. Нарушения производственного процесса
    - Запрос 
    
        ```
        Filter issues: filter = "Core Stream - All tasks"
        Insert structure Производственные метрики
        Group by Stream Sbergile
        Group by Team
        Sort by Issue Type
        Insert issues: status != cancelled and (resolved >= -30d OR statusCategory != Done ) and filter = "X-Производственные метрики-X"     
        ```
      
2. Задачи без привязки к SCP

    - Запрос 
    
        ```
        Group by Team
        Insert issues: filter = "Core Stream - All tasks" AND type in ("Change Request", Task, Epic, Story) AND (resolved >= '2022-07-01' OR statusCategory != Done) and fixVersion in versionMatchRegex("r/.*") AND status not in (Cancelled) AND project not in (SCP) AND NOT ( issueFunction in linkedIssuesOfAllRecursive("project = SCP and issuetype = Epic", "consist of ") or issueFunction in linkedIssuesOfAllRecursive("project = SCP and issuetype = Epic", "is Epic of") or type in (Epic) and "Тип Epic" = "Run / Operations / Группировка задач" or issueFunction in linkedIssuesOf("project not in (SCP) and issuetype = Epic and 'Тип Epic' = 'Run / Operations / Группировка задач' and not issueFunction in linkedIssuesOf('project in (SCP) and type in (Epic)', 'consist of ') ", "consist of ") or issueFunction in linkedIssuesOf("project not in (SCP) and issuetype = Epic and 'Тип Epic' = 'Run / Operations / Группировка задач' and not issueFunction in linkedIssuesOf('project in (SCP) and type in (Epic)', 'consist of ') ", "is Epic of") or IssueFunction in linkedIssuesOf("project not in (SCP) and issuetype = Epic AND issueFunction in linkedIssuesOfAllRecursive('project = SCP and issuetype = Epic', 'consist of ')", "is Epic of") )
        ```
      
3. Задачи закрытые в будущем

    - Запрос

        ```
        Group by Team
        Group by Fix Version/s
        Insert issues: filter = "Core Stream - All tasks" AND statusCategory = Done and (fixVersion in releaseDate("after 31d") )
        ```   

4. Out of date

    - Запрос

        ```
        Group by Team
        Insert issues: filter = "Core Stream - out-of-date issues"
        ```   

5. Задачи, которые зависли
    
    - Запрос

        ```
        Group by Team
        Group by Status
        Insert issues: filter = "Core Stream - All tasks" and type in (Bug, Incident) and project not in ("Service Management") and (status in (NeedTest, "Need Merge", "Merge Branch", "Need Info", Reopened, "in QA") and "Time in Status" in realTime(">", 24h)) and fixVersion in versionMatch("r/*") and not (issueFunction in linkedIssuesOf('statusCategory != Done', 'blocks'))
        ```   
      
    - Дополнительно 
        * Должно быть так. Но так фильтр очень сильно тормозит и ломает структуру, поэтому сделал чуть попроще НидТест=1, ИнКуА=3, НидМерж=1, МержБранч=1, ИФТ=0, НидИнфо=1, ИнРевью=1, РеОпен+Хай/Крит/Блокер=1

6. (драфт) Out of date v2

    - Запрос

        ```
        Group by Team
        Insert issues: filter = "Core Stream - out-of-date issues v2"
        ```   

7. (драфт) Открытые SubTasks при закрытом родителе

    - Запрос

        ```
        Group by Team
        Insert issues: filter = "Core Stream - All tasks" AND statusCategory != Done AND type in (subTaskIssueTypes(), Sub-task) and issueFunction in subtasksOf("statusCategory=Done")
        ```   
      
      
(далее - некритичные нарушения)

1. Задачи неправильного типа

    - Запрос

        ```
        Group by Team
        Group by Issue Type
        Insert issues: filter = "Core Stream - All tasks" AND statusCategory != Done and not (project in (EDU, FRS, BTC) and type in ("Change Request", Epic, Story, Bug, Incident) or project in (KEY) and type in ("Change Request", Epic, Story, Bug, Incident, Task) or project in ( CORE) and type in ("Change Request", Epic, Story, Task, Bug, Incident) or project in (SM) and type in (Риск, "Release 2.0") or project in (SCP) and type in (Epic) or issueFunction in subtasksOf("issuetype IN (Story, Task)"))
        ```

2. (драфт) Задачи с неправильными связями к/из SCP

    - Запрос

        ```
        Insert issues: filter = "Core Stream - All tasks" and ((project = CORE and type in (Epic) and summary ~ 'ДОД' and not issueFunction in linkedIssuesOf("project=SCP and statusCategory != Done", "consist of ")) or (project =SCP and issueFunction in linkedIssuesOf("not (project =CORE and type in (Epic) and summary ~ 'ДОД')", "is part of") and statusCategory != Done))
        ```   

3. Задачи без версии

    - Запрос

        ```
        Group by Team
        Insert issues: filter = "Core Stream - All tasks" AND statusCategory != Done AND type not in (Bug, Incident, Риск, Epic, Story, "Change Request", "Release 2.0") and not (not fixVersion is EMPTY and (fixVersion in versionMatchRegex("r/.*|#20.*")))
        ```   

4. Задачи без исполнителя

    - Запрос

        ```
        Group by Team
        Group by Fix Version/s
        Insert issues: filter = "Core Stream - All tasks" AND statusCategory != Done AND type not in (Bug, Incident, Epic, Story) and assignee is EMPTY
        ```   

5. Задачи закрытые в неправильной версии

    - Запрос

        ```
        Group by Team
        Group by Fix Version/s
        Insert issues: filter = "Core Stream - All tasks" AND statusCategory = Done and type not in (ЗНО, Риск) and project not in (SCP) AND not ( not fixVersion is EMPTY and (fixVersion in (archivedVersions(), releasedVersions(), earliestUnreleasedVersionByReleaseDate(EDU), earliestUnreleasedVersionByReleaseDate(KEY), earliestUnreleasedVersionByReleaseDate(CORE)) or fixVersion in versionMatch("r/.*") or fixVersion in versionMatch("#202.-.*") ))
        ```   

6. (драфт) Задачи без тестирования 

    - Запрос 
    
        ```
        Group by Fix Version/s
        Group by Team
        Insert issues: filter = "Core Stream - All tasks" AND created > 2022-07-01 AND status was in ('in QA')AND status not in (Cancelled) AND (type not in (subTaskIssueTypes()) AND assignee was not in team('SC Core QA') OR type  in (subTaskIssueTypes()) and issueFunction in parentsOf(" status was in ('in QA') AND status not in (Cancelled) AND assignee not in team('SC Core QA')")) ORDER BY issuetype DESC
        ```

7. Задачи с неправомерным флагом "Без изменения исходного кода"

    - Запрос

        ```
        Group by Team
        Insert issues: filter = "Core Stream - All tasks" and type in (Story) and "Без изменения исходного кода" in (Да) and created > 2022-08-01
        ```   

(далее - проблемы планирования)

1. Задачи с некорректными формулировками https://confluence.pcbltools.ru/confluence/pages/viewpage.action?pageId=76683568

    - Запрос

        ```
        Group by Team
        Insert issues: filter = "Core Stream - All tasks" AND type in (Epic) and project in (CORE) and summary ~ 'ДОД' AND (NOT (summary ~ 'Реализован*' OR summary ~ 'Проработан*' OR summary ~ 'внедрен*' OR summary ~ 'удален*' OR summary ~ 'настроен*' OR summary ~ "\\[RoadMap\\]" OR summary ~ "\\[Backlog\\]") OR summary ~ Поддержка OR summary ~ Стабилизация OR summary ~ Развитие OR summary ~ Исправление OR summary ~ Исправление OR summary ~ Актуализация OR summary ~ Техдолг OR summary ~ Оптимизация OR summary ~ Повышение) ORDER BY fixVersion ASC
        ```   

2. (драфт) Ошибки планирования

    - Запрос

        ```
        Group by Team
        Insert issues: filter = "Core Stream - All tasks" AND type in ("Change Proposal", "Change Request", Epic, Story, Task) and (fixVersion in ("#2023q1", "#2023q2", "#2023q3", "#2023q4") and (priority not in (Lowest) or status not in (open) or issueFunction in linkedIssuesOfRemote("query", "pageId=76679613") ) or ( fixVersion in ( "#2022q4", "r/2022-10", "r/56.0.0-rc1", "r/57.0.0-rc1", "r/58.0.0-rc1", "r/59.0.0-rc1", "r/60.0.0-rc1", "r/48.0.0-rc1", "r/48.1.0-rc1" ) and not issueFunction in linkedIssuesOfRemote("query", "pageId=76679613") ) ) and not (fixVersion in ("r/48.0.0-rc1", "r/48.1.0-rc1") and project in ( EDU, SM)) and assignee not in (MVMilkov)
        ```   

3. Несогласованные цели

    - Запрос 
    
        ```
        Group by Fix Version/s    
        Insert issues: filter = "Core Stream - All goals and DoDs" and (Verified is EMPTY )
        ```
    
Метрики
-----------

* TBD 

Остальное
-----------

* Плагины 
    - Script Runner
    - Time in status
    - Approvals / DoR / DoD
    - SLA Indicator
* Настройки 
    - Tempo Team 
    - Workflow
        * Общее 
            - Team (выбор из списка)
            - Stream (автоматическое вычисление из поля Team)
        * Epic
            - Добавлена связь is part of / consist of
        * Change Request 
        * Story 
            - добавлены поля для DoR / DoD (см чеклист деливери лида)
        * Task 
        * SubTask
            - наследует fixVersion от Story
        * Bug 
            - Добавлено поле "Основания допуска дефекта в прод"
            - Добавлено поле "Корневые причины возникновения дефекта"
            - Добавлено поле "Меры по недопущению подобных инцидентов в будущем"
            - Добавлено поле "Причины возникновения" (выбор нескольких значений из списка)
                * TBD
            - Добавлено поле "Аллокация" (выбор нескольких значений из списка)
        * Incident
            - Добавлено поле "Корневые причины возникновения дефекта"
            - Добавлено поле "Меры по недопущению подобных инцидентов в будущем"
            - Добавлено поле "Причины возникновения" (выбор нескольких значений из списка)
            - Добавлена связь Причина в / Следствие
        * Меры по недопущению подобных инцидентов в будущем
        * Причины возникновения
            - TBD
        * Риски
    - components
        * внутренние 
        * внешние
* Процессы  
    - SLA / Incidents
        * 
    - Риски 
        * 
    - Bug / Инцидент
        * RCA
        * Запрет релиза при неперенесенных задачах
    - Story delivery
        * см чеклист деливери лида


Критерии переходов задач между статусами