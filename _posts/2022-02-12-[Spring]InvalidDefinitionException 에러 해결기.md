---
layout: post
categories: 스프링 오류
---

컨트롤러 테스트 도중 
```console
Cannot construct instance of `java.time.LocalDateTime` (no Creators, like default constructor, exist): no String-argument constructor/factory method to deserialize from String value ('2022-02-12T17:32:46.4275326')
 at [Source: (String)"{"createdDate":"2022-02-12T17:32:46.4275326","modifiedDate":"2022-02-12T17:32:46.4275326","id":1,"title":"테스트 제목","content":"테스트 내용","author":"테스트 저자"}"; line: 1, column: 16] (through reference chain: com.kyuwon.booklog.domain.posts.Posts["createdDate"])
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `java.time.LocalDateTime` (no Creators, like default constructor, exist): no String-argument constructor/factory method to deserialize from String value ('2022-02-12T17:32:46.4275326')
 at [Source: (String)"{"createdDate":"2022-02-12T17:32:46.4275326","modifiedDate":"2022-02-12T17:32:46.4275326","id":1,"title":"테스트 제목","content":"테스트 내용","author":"테스트 저자"}"; line: 1, column: 16] (through reference chain: com.kyuwon.booklog.domain.posts.Posts["createdDate"])
	at com.fasterxml.jackson.databind.exc.InvalidDefinitionException.from(InvalidDefinitionException.java:67)
	at com.fasterxml.jackson.databind.DeserializationContext.reportBadDefinition(DeserializationContext.java:1615)
	at com.fasterxml.jackson.databind.DatabindContext.reportBadDefinition(DatabindContext.java:400)
	at com.fasterxml.jackson.databind.DeserializationContext.handleMissingInstantiator(DeserializationContext.java:1077)
	at com.fasterxml.jackson.databind.deser.ValueInstantiator._createFromStringFallbacks(ValueInstantiator.java:371)
	at com.fasterxml.jackson.databind.deser.std.StdValueInstantiator.createFromString(StdValueInstantiator.java:323)
	at com.fasterxml.jackson.databind.deser.BeanDeserializerBase.deserializeFromString(BeanDeserializerBase.java:1408)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer._deserializeOther(BeanDeserializer.java:176)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:166)
	at com.fasterxml.jackson.databind.deser.impl.FieldProperty.deserializeAndSet(FieldProperty.java:138)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.vanillaDeserialize(BeanDeserializer.java:293)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:156)
	at com.fasterxml.jackson.databind.ObjectMapper._readMapAndClose(ObjectMapper.java:4526)
	at com.fasterxml.jackson.databind.ObjectMapper.readValue(ObjectMapper.java:3468)
	at com.fasterxml.jackson.databind.ObjectMapper.readValue(ObjectMapper.java:3436)
	at com.kyuwon.booklog.controller.posts.PostsControllerTest.preparePost(PostsControllerTest.java:297)
	at com.kyuwon.booklog.controller.posts.PostsControllerTest.access$100(PostsControllerTest.java:44)
  (생략)
```
다음과 같은 에러가 났다. 

- `com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `java.time.LocalDateTime` (no Creators, like default constructor, exist)`
  - 과연 이 에러가 뭘까 ?
  - `LocalDateTime`을 JSON을 파싱한 결과를 전달할 적절한 생성자를 찾지 못했을 때 발생하는 에러이다. 
- 해결방법으로는 여러가지가 있었는데 그중에 `JacksonConfiguration`을 생성하여 `LocalDateTime`을 직렬화 하는 방법이 있었는데 결과는 여전히 에러.

### JacksonConfiguration

```java
package com.kyuwon.booklog.config;

import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JsonDeserializer;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializerProvider;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import java.io.IOException;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

/**
 * LocalDate를 사용하여 Json 통신을 하기 위한 Jackson 설정 파일
 */
@Configuration
public class JacksonConfiguration {
    public static final DateTimeFormatter FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    /**
     * 날짜 객체와 JSON 매핑을 해준다.
     * JavaTimeModule에 날짜 직렬화, 역직렬화를 설정해주고 objectMapper에 모듈을 설정한 뒤 리턴해준다.
     *
     * @return 객체
     */
    @Bean
    @Primary
    public ObjectMapper serializingObjectMapper() {
        ObjectMapper objectMapper = new ObjectMapper();
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        javaTimeModule.addSerializer(LocalDate.class, new LocalDateSerializer());
        javaTimeModule.addDeserializer(LocalDate.class, new LocalDateDeserializer());
        objectMapper.registerModule(javaTimeModule);
        return objectMapper;
    }

    /**
     * 날짜 개체 또는 값 형식을 JSON으로 직렬화하는 기능을 제공한다.
     */
    public class LocalDateSerializer extends JsonSerializer<LocalDate> {
        /**
         * 날짜 객체를 JSON 문자열로 반환한다.
         *
         * @param value       변환할 날짜 값
         * @param gen         JSON 생성자
         * @param serializers 직렬화 제공자
         * @throws IOException
         */
        @Override
        public void serialize(LocalDate value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
            gen.writeString(value.format(FORMATTER));
        }
    }

    /**
     * 날짜 JSON을 개체 또는 값 형식으로 역직렬화하는 기능을 제공한다.
     */
    public class LocalDateDeserializer extends JsonDeserializer<LocalDate> {
        /**
         * JSON 값을 날짜 인스턴스로 나타낸다.
         *
         * @param parser JSON 파싱해주는 객체
         * @param ctxt   역직렬화
         * @return 파싱된 날짜
         * @throws IOException
         * @throws JsonProcessingException
         */
        @Override
        public LocalDate deserialize(JsonParser parser, DeserializationContext ctxt) throws IOException, JsonProcessingException {
            return LocalDate.parse(parser.getValueAsString(), FORMATTER);
        }
    }
}
```

- 다시 해결방법을 찾다가 `objectMapper`에 `JavaTimeModule()`을 추가해주니깐 해결되었다. 
  - `objectMapper.registerModule(new JavaTimeModule());`

- 위 `configuration`에서도 `JavaTimeModule`을 설정해주었는데 왜 작동이 안되었을까? 
- 아무래도 `objectMapper` 생성시 `JacksonConfiguration`를 설정하지 않아서 인거 같다. 
