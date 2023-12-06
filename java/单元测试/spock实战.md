
```java
package cn.xx.manage.service.impl

import cn.xx.common.dto.additional.ServiceAdditionalPageDTO
import cn.x.common.entity.ServiceAdditional
import cn.x.common.exception.ApiErrorCode
import cn.x.common.exception.ApiException
import cn.x.common.mapper.ServiceAdditionalMapper
import org.dbunit.DbUnit
import org.dbunit.supper.MyBaseSpec

class ServiceAdditionalServiceImplTest extends MyBaseSpec {

    private ServiceAdditionalServiceImpl serviceAdditionalService

    void setup() {
        serviceAdditionalService = new ServiceAdditionalServiceImpl()
        serviceAdditionalService.serviceAdditionalMapper = MapperUtil.getMapper(ServiceAdditionalMapper)
    }

    @DbUnit(content = {
        service_additional(
                "service_additional_id": "1",
                "service_name": "123",
                "skill_one_id": "1",
                "skill_two_id": "2",
                "skill_three_id": "3",
                "service_type": "10",
                "service_additional_amount": "20",
                "worker_service_additional_amount": "0"
        )
    })
    def "保存新数据(test save)"() {
        given:
        def additional = new ServiceAdditional(serviceName: "345", skillOneId: 1, skillTwoId: 2, skillThreeId: 3, serviceType: 10, serviceAdditionalAmount: 10, workerServiceAdditionalAmount: 10)
        when:
        serviceAdditionalService.save(additional)
        then:
        def serviceAdditional = serviceAdditionalService.serviceAdditionalMapper.selectById(2)
        serviceAdditional.serviceName == additional.serviceName
    }

    @DbUnit(content = {
        service_additional(
                "service_additional_id": "1",
                "service_name": "123",
                "skill_one_id": "1",
                "skill_two_id": "2",
                "skill_three_id": "3",
                "service_type": "10",
                "service_additional_amount": "20",
                "worker_service_additional_amount": "0"
        )
    })
    def "保存新数据(test save)-边界测试"() {
        given:
        def additional = new ServiceAdditional(serviceName: serviceName, skillOneId: skillOneId, skillTwoId: skillTwoId, skillThreeId: skillThreeId, serviceType: 10, serviceAdditionalAmount: 10, workerServiceAdditionalAmount: 10)
        when:
        serviceAdditionalService.save(additional)
        then:
        def e = thrown(ApiException)
        e.getMessage() == message
        where:
        serviceName | skillOneId | skillTwoId | skillThreeId || message
        "123"       | 1          | 2          | 3            || ApiErrorCode.REPEAT_SERVICE_NAME.getMessage()
    }

    @DbUnit(content = {
        service_additional(
                "service_additional_id": "1",
                "service_name": "123",
                "skill_one_id": "1",
                "skill_two_id": "2",
                "skill_three_id": "3",
                "service_type": "10",
                "service_additional_amount": "20",
                "worker_service_additional_amount": "0"
        )
        service_additional(
                "service_additional_id": "2",
                "service_name": "124",
                "skill_one_id": "1",
                "skill_two_id": "2",
                "skill_three_id": "3",
                "service_type": "10",
                "service_additional_amount": "20",
                "worker_service_additional_amount": "0"
        )
    })
    def "修改数据(test save)-边界测试"() {
        given:
        def additional = new ServiceAdditional(serviceAdditionalId: serviceAdditionalId, serviceName: serviceName, skillOneId: skillOneId, skillTwoId: skillTwoId, skillThreeId: skillThreeId, serviceType: 10, serviceAdditionalAmount: 10, workerServiceAdditionalAmount: 10)
        when:
        serviceAdditionalService.save(additional)
        then:
        def e = thrown(ApiException)
        e.getMessage() == message
        where:
        serviceAdditionalId | serviceName | skillOneId | skillTwoId | skillThreeId || message
        1                   | "124"       | 1          | 2          | 3            || ApiErrorCode.REPEAT_SERVICE_NAME.getMessage()
    }

    @DbUnit(content = {
        service_additional(
                "service_additional_id": "1",
                "service_name": "123",
                "skill_one_id": "1",
                "skill_two_id": "2",
                "skill_three_id": "3",
                "service_type": "10",
                "service_additional_amount": "20",
                "worker_service_additional_amount": "0"
        )
        service_additional(
                "service_additional_id": "2",
                "service_name": "124",
                "skill_one_id": "1",
                "skill_two_id": "2",
                "skill_three_id": "3",
                "service_type": "10",
                "service_additional_amount": "20",
                "worker_service_additional_amount": "0"
        )
    })
    def "修改数据(test save)-正常修改"() {
        given:
        def additional = new ServiceAdditional(serviceAdditionalId: serviceAdditionalId, serviceName: serviceName, skillOneId: skillOneId, skillTwoId: skillTwoId, skillThreeId: skillThreeId, serviceType: 10, serviceAdditionalAmount: 10, workerServiceAdditionalAmount: 10)
        when:
        serviceAdditionalService.save(additional)
        then:
        def additionalT = serviceAdditionalService.serviceAdditionalMapper.selectById(serviceAdditionalId)
        additionalT.getServiceName() == serviceName
        where:
        serviceAdditionalId | serviceName | skillOneId | skillTwoId | skillThreeId || message
        1                   | "123-新"    | 1          | 2          | 3            || ApiErrorCode.REPEAT_SERVICE_NAME.getMessage()
    }

    @DbUnit(content = {
        service_additional(
                "service_additional_id": "1",
                "service_name": "123",
                "skill_one_id": "1",
                "skill_two_id": "2",
                "skill_three_id": "3",
                "service_type": "10",
                "service_additional_amount": "20",
                "worker_service_additional_amount": "0"
        )
    })
    def "test del"() {

        when:
        serviceAdditionalService.del(1)
        then:
        def serviceAdditional = serviceAdditionalService.serviceAdditionalMapper.selectById(1)
        serviceAdditional == null
    }

    @DbUnit(content = {
        service_additional(
                "service_additional_id": "1",
                "service_name": "123",
                "skill_one_id": "1",
                "skill_two_id": "2",
                "skill_three_id": "3",
                "service_type": "10",
                "service_additional_amount": "20",
                "worker_service_additional_amount": "0"
        )
    })
    def "查询列表(test list)-只有分页数据"() {
        given:
        def dto = new ServiceAdditionalPageDTO(page: 1, limit: 10)
        when:
        def pageData = serviceAdditionalService.list(dto)
        then:
        pageData.getTotal() > 0
    }

    @DbUnit(content = {
        service_additional(
                "service_additional_id": "1",
                "service_name": "123",
                "skill_one_id": "1",
                "skill_two_id": "2",
                "skill_three_id": "3",
                "service_type": "10",
                "service_additional_amount": "20",
                "worker_service_additional_amount": "0"
        )
    })
    def "查询列表(test list)-有分页、技能相关等数据"() {
        given:
        def dto = new ServiceAdditionalPageDTO(page: 1, limit: 10, skillOneId: 1, skillTwoId: 2, skillThreeId: 3, serviceType: 10)
        when:
        def pageData = serviceAdditionalService.list(dto)
        then:
        pageData.getTotal() > 0
    }

    @DbUnit(content = {
        service_additional(
                "service_additional_id": "1",
                "service_name": "123",
                "skill_one_id": "1",
                "skill_two_id": "2",
                "skill_three_id": "3",
                "service_type": "10",
                "service_additional_amount": "20",
                "worker_service_additional_amount": "0"
        )
    })
    def "查询列表(test list)-有分页、技能相关等数据-没查询到数据"() {
        given:
        def dto = new ServiceAdditionalPageDTO(page: 1, limit: 10, skillOneId: 1, skillTwoId: 2, skillThreeId: 4, serviceType: 10)
        when:
        def pageData = serviceAdditionalService.list(dto)
        then:
        pageData.getTotal() == 0
    }

    @DbUnit(content = {
        service_additional(
                "service_additional_id": "1",
                "service_name": "123",
                "skill_one_id": "1",
                "skill_two_id": "2",
                "skill_three_id": "3",
                "service_type": "10",
                "service_additional_amount": "20",
                "worker_service_additional_amount": "0"
        )
        service_additional(
                "service_additional_id": "2",
                "service_name": "安装",
                "skill_one_id": "1",
                "skill_two_id": "2",
                "skill_three_id": "3",
                "service_type": "10",
                "service_additional_amount": "20",
                "worker_service_additional_amount": "0"
        )
    })
    def "查询列表(test list)-有分页、服务名称"() {
        given:
        def dto = new ServiceAdditionalPageDTO(page: 1, limit: 10, serviceType: 10, serviceName: "2")
        when: "服务名称模糊查询到数据"
        def pageData = serviceAdditionalService.list(dto)
        then:
        pageData.getTotal() == 1
    }

    @DbUnit(content = {
        service_additional(
                "service_additional_id": "1",
                "service_name": "123",
                "skill_one_id": "1",
                "skill_two_id": "2",
                "skill_three_id": "3",
                "service_type": "10",
                "service_additional_amount": "20",
                "worker_service_additional_amount": "0"
        )
        service_additional(
                "service_additional_id": "2",
                "service_name": "安装",
                "skill_one_id": "1",
                "skill_two_id": "2",
                "skill_three_id": "3",
                "service_type": "10",
                "service_additional_amount": "20",
                "worker_service_additional_amount": "0"
        )
        Skill(skill_id: 1, skill_name: "第一级", parent_id: 0, level: 1)
        Skill(skill_id: 2, skill_name: "第二级", parent_id: 1, level: 2)
        Skill(skill_id: 3, skill_name: "第三级", parent_id: 2, level: 3)

    })
    def "查询列表(test list)-有分页、无服务名称"() {
        given:
        def dto = new ServiceAdditionalPageDTO(page: 1, limit: 10, serviceType: 10)
        when: "无服务名称-不进行模糊查询到数据"
        def pageData = serviceAdditionalService.list(dto)
        then:
        pageData.getTotal() == 2
    }


}

```
这套spock除了没有使用到spring框架，其余的都使用了<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/2923644/1688433395809-b29e8162-d4e6-4dfc-af52-b87add8eb131.png#averageHue=%233e4346&clientId=u7c2ffcd5-fffd-4&from=paste&height=222&id=u24aaa6cd&originHeight=333&originWidth=604&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=35970&status=done&style=none&taskId=ub6276438-89b6-4d7a-bc1c-a4f1b9053d2&title=&width=402.6666666666667)<br />选择覆盖率运行(需要查看代码覆盖率的时候)<br />选择直接运行(直接运行测试单元代码)
