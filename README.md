@Configuration
@ConfigurationProperties(prefix = "dsl")
@Data
public class DSLProperties {
  @NestedConfigurationProperty
  private Map<String, Map<String, API>> apps;
  @NestedConfigurationProperty
  private BaselinePerformanceRequirement baselinePerfRequirement;

  @Data
  public static class PerformanceRequirementLatencyCriteria {
    @DurationUnit(ChronoUnit.MILLIS)
    private Duration p95;
  }

  @Data
  public static class BaselinePerformanceRequirement {
    private String atThroughput;
    private PerformanceRequirementLatencyCriteria yieldsLatencyLessThan;
    private String errPercentLessThan;
  }


  @Data
  public class App {
    @NestedConfigurationProperty
    private Map<String, API> apis;


    @Data
    public class API {
      @NestedConfigurationProperty
      private APIInfo withApiInfo;
      @NestedConfigurationProperty
      private PerformanceRequirement meetsPrefRequirement;

      @Data
      public class APIInfo {
        private String uri;
        private String method;
        private String body;
      }

      @Data
      public class PerformanceRequirement {
        private String atThroughput;
        @NestedConfigurationProperty
        private PerformanceRequirementLatencyCriteria yieldsLatencyLessThan;
        private String errPercentLessThan;
        private String baselinePerfRequirementValue;

//        @ConstructorBinding
        public PerformanceRequirement(String atThroughput, PerformanceRequirementLatencyCriteria yieldsLatencyLessThan, String errPercentLessThan, String baselinePerfRequirementValue) {
          if (baselinePerfRequirementValue != null) {
            setExpandedValuesFromBaselineShorthand(DSLProperties.this.baselinePerfRequirement, baselinePerfRequirementValue);
          } else {
            this.atThroughput = atThroughput;
            this.yieldsLatencyLessThan = yieldsLatencyLessThan;
            this.errPercentLessThan = errPercentLessThan;
          }
        }

        private void setExpandedValuesFromBaselineShorthand(BaselinePerformanceRequirement baselinePerformanceRequirement, String baselineMultiplier) {
          this.atThroughput = baselinePerformanceRequirement.getAtThroughput();
          this.yieldsLatencyLessThan = baselinePerformanceRequirement.getYieldsLatencyLessThan();
          this.errPercentLessThan = baselinePerformanceRequirement.getErrPercentLessThan();
        }

        public void setBaselinePerfRequirementValue(String baselinePerfRequirementValue) {
          this.baselinePerfRequirementValue = baselinePerfRequirementValue;
        }
      }
    }
  }


}




dsl:
  baseline-perf-requirement:
    at-throughput: 20 req/sec
    yields-latency-less-than:
      p95: 200ms
    err-percent-less-than: 10%
  apps:
    alerts:
      get-alerts:
        with-api-info:
          uri: /alerts
          method: GET
        meets-pref-requirement:
          at-throughput: 20 req/sec
          yields-latency-less-than:
            p95: 1s
          err-percent-less-than: 10%
    bill-pay:
      update-recipient:
        with-api-info:
          uri: '/billpay/recipients/${RECIPIENT_ID}'
          method: PUT
          body: |
            {
              "name": "${NAME}",
              "nickname": "Duke Energy - NC & SC",
              "accountNumber": "${ACCOUNT_NUMBER}"
            }
        meets-pref-requirement:
          baseline-perf-requirement-value: 1x #TODO: support baseline injection
      get-billpay:
        with-api-info:
          uri: /billPay
          method: GET
        meets-pref-requirement:
          at-throughput: 100 req/sec
          yields-latency-less-than:
            p95: 10ms
          err-percent-less-than: 5%
      add-recipients:
        with-api-info:
          uri: /recipients
          method: POST
        meets-pref-requirement:
          at-throughput: 20 req/sec
          yields-latency-less-than:
            p95: 1500ms
          err-percent-less-than
          
 

