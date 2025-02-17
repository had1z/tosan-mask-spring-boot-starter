# tosan-mask-spring-boot-starter

This project provides an Spring-Boot Starter that provides masking sensitive data in json string functionality.

## Usage

The `tosan-mask-spring-boot-starter` brings most of the required configuration with it, therefore you only need to add
it as a maven dependency and enable the desired functionality. with this library you can define your desired mask style,
and it will be registered in library automatically.

```
<dependency>
  <groupId>com.tosan.tools</groupId>
  <artifactId>tosan-mask-spring-boot-starter</artifactId>
  <version>latest-version</version>
</dependency>
```

in order to register additional mask styles you have to define SecureParametersConfig bean. this bean has been defined
by default as below:

```
    public static final Set<SecureParameter> SECURED_PARAMETERS = new HashSet<SecureParameter>() {
        {
            add(new SecureParameter("password", MaskType.COMPLETE));
            add(new SecureParameter("pan", MaskType.PAN));
            add(new SecureParameter("pin", MaskType.COMPLETE));
            add(new SecureParameter("cvv2", MaskType.SEMI));
            add(new SecureParameter("expDate", MaskType.SEMI));
            add(new SecureParameter("username", MaskType.SEMI));
            add(new SecureParameter("mobile", MaskType.MOBILE));
        }
    };

    @Bean
    @ConditionalOnMissingBean
    public SecureParametersConfig secureParametersConfig() {
        return new SecureParametersConfig(SECURED_PARAMETERS);
    }

```

above configuration means for example if a field named 'pan' (equalsIgnoreCase) exist in json string the value will be masked as PAN mask
style. in order to apply mask types you can inject ReplaceHelperDecider and use replace method as below:

```
@Componenet
public class Replacer {

    @Autowired
    private JsonReplaceHelperDecider jsonReplaceHelperDecider;

    public void testMask() {
        String replace = jsonReplaceHelperDecider.replace("{\"pan\":\"5022291075648374\", \"password\":\"1234\"}");
        System.out.println(replace);
    }
}
```

input json:

```
{
   "pan":"5022291075648374",
   "password":"1234"
}
```

output json:

```
{
   "pan":"*SEMI_ENCRYPTED:502229******8374",
   "password":"ENCRYPTED"
}
```

default mask types work as below:
> MaskType.COMPLETE : mask value completely(ex: "ENCRYPTED" for initial value="testValue")

> MaskType.PAN : mask pan data, mask middle characters(ex: "*SEMI_ENCRYPTED:502229******8374")

> MaskType.LEFT : mask left part of string(ex: "*SEMI_ENCRYPTED:Value" for initial value="testValue")

> MaskType.RIGHT : mask right part of string(ex: "*SEMI_ENCRYPTED:test" for initial value="testValue")

> MaskType.SEMI : mask 5 last characters for strings more than 10 char length and mask half for less than 10 char strings

> MaskType.MOBILE : mask 3 middle characters of a mobile number (e.g. `*SEMI_ENCRYPTED:0911***4506`) 

JsonReplaceHelperDecider bean uses two other beans named JacksonReplaceHelper and RegexReplaceHelper beans to mask
strings. in such a way that it tries to use JacksonReplaceHelper first. in order to use this kind of replaceHelper no
exception must happen while parsing json string with jackson library. if JsonProcessingException happen in string
parsing, replacer will be switched to RegexReplaceHelper. this ReplaceHelper tries to mask values with regex pattern.
each one of below ReplaceHelpers can be injected in code and be used separately as desired. 

> JacksonReplaceHelper

> RegexReplaceHelper

replaceHelperDecider have different methods that use these helpers separately.

### extending mask types

all default mask types has been defined as ConditionalMissingBeans so the behaviour of each type can be overridden with
your preferred behaviour. in order to define new mask type, below procedure must be followed:

1. define new mask type:

```
public class UserMaskType extends MaskType {

   public static final MaskType TEST_MASK_TYPE = new MaskType();
   public static final MaskType SECRET_MASK_TYPE = new MaskType();
}
```

2. define mask behaviour related to new mask type:

```
@Component
public class TestValueMasker implements ValueMasker {
    @Override
    public MaskType getType() {
        return UserMaskType.TEST_MASK_TYPE;
    }

    @Override
    public String mask(String parameterPlainValue) {
        return "test";
    }
}
```

3. define SecureParametersConfig:

```
    @Bean
    @Primary
    public SecureParametersConfig secureParametersConfig() {
        Set<SecureParameter> securedParameters = new HashSet<SecureParameter>() {
            {
                add(new SecureParameter("pan", UserMaskType.PAN));
                add(new SecureParameter("testField", UserMaskType.TEST_MASK_TYPE));
            }
        };
        return new SecureParametersConfig(securedParameters);
    }
```

attentions: after overriding SecureParametersConfig only elements defined in map will be applied 
and default mappings are not considered. in order to apply default mask mappings you can define 
your bean as below:

```
    @Bean
    public SecureParametersConfig securedParametersWithDefault() {
        Set<SecureParameter> securedParameters = MaskBeanConfiguration.SECURED_PARAMETERS;
        securedParameters.add(new SecureParameter("testField", UserMaskType.TEST_MASK_TYPE));
        return new SecureParametersConfig(securedParameters);
    }
```

attentions: field name checking is currently comparing the exact filed name with equalsIgnoreCase method. 
this feature will be expanded to more options in the future.

### adding comparisonType to secureParameter
from version 1.0.4 you can define comparison type for each secure parameter element. this types consist of:

> EQUALS

> EQUALS_IGNORE_CASE

> LIKE

> RIGHT_LIKE

> LEFT_LIKE

if you don't specify any comparison type EQUALS_IGNORE_CASE type will be selected and applied. these types
indicate the operator comparing fieldName and parameterName and can be defined in your configuration as below:

```
    @Bean
    public SecureParametersConfig securedParametersWithComparisonType() {
        Set<SecureParameter> securedParameters = new HashSet<>();
        securedParameters.add(new SecureParameter("pan", MaskType.PAN, ComparisonType.RIGHT_LIKE));
        return new SecureParametersConfig(securedParameters);
    }

```

### Prerequisites
This Library requires java version 17 or above and spring boot version 3 and above.

## Contributing
Any contribution is greatly appreciated.

If you have a suggestion that would make this project better, please fork the repo and create a pull request.
You can also simply open an issue with the tag "enhancement".

## License
The source files in this repository are available under the [Apache License Version 2.0](./LICENSE.txt).
