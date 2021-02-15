# Pipeline Examples
 
## ExtractAndSumScore class definition

```java
public static class ExtractAndSumScore
    extends PTransform<PCollection<GameActionInfo>, PCollection<KV<String, Integer>>> {

  private final String field;

  ExtractAndSumScore(String field) {
    this.field = field;
  }

  @Override
  public PCollection<KV<String, Integer>> expand(PCollection<GameActionInfo> gameInfo) {

    return gameInfo
        .apply(
            MapElements.into(
                    TypeDescriptors.kvs(TypeDescriptors.strings(), TypeDescriptors.integers()))
                .via((GameActionInfo gInfo) -> KV.of(gInfo.getKey(field), gInfo.getScore())))
        .apply(Sum.integersPerKey());
  }
}
```

## Basic ETL pipeline no windowing nothing

```java
public static void main(String[] args) throws Exception {
  // Begin constructing a pipeline configured by commandline flags.
  Options options = PipelineOptionsFactory.fromArgs(args).withValidation().as(Options.class);
  Pipeline pipeline = Pipeline.create(options);

  // Read events from a text file and parse them.
  pipeline
      .apply(TextIO.read().from(options.getInput()))
      .apply("ParseGameEvent", ParDo.of(new ParseEventFn()))
      // Extract and sum username/score pairs from the event data.
      .apply("ExtractUserScore", new ExtractAndSumScore("user"))
      .apply(
          "WriteUserScoreSums", new WriteToText<>(options.getOutput(), configureOutput(), false));

  // Run the batch pipeline.
  pipeline.run().waitUntilFinish();
}
```


## Running pipelines with FIXED - WINDOW of 1 hour (Game example)

```java
public static void main(String[] args) throws Exception {
  // Begin constructing a pipeline configured by commandline flags.
  Options options = PipelineOptionsFactory.fromArgs(args).withValidation().as(Options.class);
  Pipeline pipeline = Pipeline.create(options);

  final Instant stopMinTimestamp = new Instant(minFmt.parseMillis(options.getStopMin()));
  final Instant startMinTimestamp = new Instant(minFmt.parseMillis(options.getStartMin()));

  // Read 'gaming' events from a text file.
  pipeline
      .apply(TextIO.read().from(options.getInput()))
      // Parse the incoming data.
      .apply("ParseGameEvent", ParDo.of(new ParseEventFn()))

      // Filter out data before and after the given times so that it is not included
      // in the calculations. As we collect data in batches (say, by day), the batch for the day
      // that we want to analyze could potentially include some late-arriving data from the
      // previous day.
      // If so, we want to weed it out. Similarly, if we include data from the following day
      // (to scoop up late-arriving events from the day we're analyzing), we need to weed out
      // events that fall after the time period we want to analyze.
      // [START DocInclude_HTSFilters]
      .apply(
          "FilterStartTime",
          Filter.by(
              (GameActionInfo gInfo) -> gInfo.getTimestamp() > startMinTimestamp.getMillis()))
      .apply(
          "FilterEndTime",
          Filter.by(
              (GameActionInfo gInfo) -> gInfo.getTimestamp() < stopMinTimestamp.getMillis()))
      // [END DocInclude_HTSFilters]

      // [START DocInclude_HTSAddTsAndWindow]
      // Add an element timestamp based on the event log, and apply fixed windowing.
      .apply(
          "AddEventTimestamps",
          WithTimestamps.of((GameActionInfo i) -> new Instant(i.getTimestamp())))
      .apply(
          "FixedWindowsTeam",
          Window.into(FixedWindows.of(Duration.standardMinutes(options.getWindowDuration()))))
      // [END DocInclude_HTSAddTsAndWindow]

      // Extract and sum teamname/score pairs from the event data.
      .apply("ExtractTeamScore", new ExtractAndSumScore("team"))
      .apply(
          "WriteTeamScoreSums", new WriteToText<>(options.getOutput(), configureOutput(), true));

  pipeline.run().waitUntilFinish();
}
```



## Implementation of stateful ParDo
```java
/**
 * Implements a streaming merge using stateful ParDo. Waits for two elements to arrive with the same key,
 * storing the first in MapState until the second arrives, then merges the two and outputs the merged element,
 * and finally clears the keyed state. This technique avoid double writes to the output database, reducing cost
 * on some target systems (e.g. RDS) and improving performance during backfill operations (where writes to the db
 * tend to bottleneck the pipeline).
 *
 * @param <K> type of key
 * @param <V> type of value
 */
public class StreamingMerge<K, V extends Message>
    extends DoFn<KV<K, V>, KV<K, V>> {

  @StateId("index")
  private final StateSpec<MapState<K, V>> indexSpec;
  private final SerializableBiFunction<V, V, V> mergeFunction;
  private final Maybe<SerializableFunction<V, Boolean>> dataCompletenessFunction;

  /**
   * This function is used to merge multiple updates coming from 1 or more streams into a single entry
   * that gets emitted when data is complete. User would define what does data completion means. State gets
   * retained in the map even after emitting a record downstream. This is really helpful in cases where
   * you want to avoid sending partial records downstream. Eg SNS notifications.
   *
   * @param keyCoder specify a coder for your key
   * @param valueCoder specify a coder for the value
   * @param mergeFunction function representing how to merge 2 records together
   * @param dataCompletenessFunction function representing data completeness
   * @param <K> key type
   * @param <V> value type
   * @return an object of StreamingMerge
   */
  public static <K, V extends Message> StreamingMerge<K, V> of(
          Coder<K> keyCoder,
          Coder<V> valueCoder,
          SerializableBiFunction<V, V, V> mergeFunction,
          Maybe<SerializableFunction<V, Boolean>> dataCompletenessFunction) {
    return new StreamingMerge<>(
        keyCoder, valueCoder, mergeFunction, dataCompletenessFunction);
  }

  /**
   * This function is used to merge exactly 2 changes to a key. If the key does not exist in the map,
   * it creates an entry in the map and updates the map when the 2nd change arrives.
   * As soon as the 2nd change arrives, the updated value is emitted downstream and the value is removed
   * from the map.
   * It uses a default function to merge two different protos. It also provides a default completeness function
   * which just lets everything pass through.
   * If your pipelines relies on having more than 2 updates for a key, you should pass in a custom data
   * completeness function that would be used to figure out when to emit something downstream.
   *
   * @param keyCoder specify a coder for your key
   * @param valueCoder specify a coder for the value
   * @param <K> key type
   * @param <V> value type
   * @return an object of StreamingMerge
   */
  @SuppressWarnings("unchecked")
  public static <K, V extends Message> StreamingMerge<K,V> of(Coder<K> keyCoder, Coder<V> valueCoder) {
    return new StreamingMerge<>(
        keyCoder,
        valueCoder,
        (V left, V right) -> (V) left.toBuilder().mergeFrom(right).build(),
        Maybe.empty());
  }

  private StreamingMerge(
      Coder<K> keyCoder, Coder<V> valueCoder,
      SerializableBiFunction<V, V, V> mergeFunction,
      Maybe<SerializableFunction<V, Boolean>> dataCompletenessFunction) {
    this.indexSpec = StateSpecs.map(keyCoder, valueCoder);
    this.mergeFunction = mergeFunction;
    this.dataCompletenessFunction = dataCompletenessFunction;
  }

  /**
   * Gets called for each element.
   * @param context context
   * @param index mapIndex
   */
  @ProcessElement
  public void processElement(
      ProcessContext context,
      @StateId("index") MapState<K, V> index) {

    final K currentElementKey = context.element().getKey();
    final ReadableState<V> currentState = index.get(currentElementKey);
    final V valueFromState = currentState.read();
    final V currentElementValue = context.element().getValue();

    if (valueFromState != null) {
      final V currentElementValueCopy = currentElementValue;
      V updateValue = mergeFunction.apply(valueFromState, currentElementValueCopy);
      // We will only emit something downstream when the completeness function specified by the user returns true
      if (dataCompletenessFunction.isPresent()) {
        index.put(currentElementKey, updateValue);
        if (dataCompletenessFunction.get().apply(updateValue)) {
          context.output(KV.of(currentElementKey, updateValue));
        }
      } else {
        index.remove(currentElementKey);
        context.output(KV.of(currentElementKey, updateValue));
      }
    } else {
      index.put(currentElementKey, currentElementValue);
    }
  }
}
```


## Json to Proto/AutoValue
```java
public class JsonToJournalRecords
    extends PTransform<PCollection<String>, PCollection<JournalRecord<JsonNode>>> {

  private static final JsonToJournalRecords INSTANCE = new JsonToJournalRecords();
  private static final ObjectMapper OBJECT_MAPPER = ObjectMapperBuilder.get();
  private static final ObjectReader OBJECT_READER = OBJECT_MAPPER.readerFor(
      new TypeReference<JournalRecord<JsonNode>[]>() {});

  public static JsonToJournalRecords of() {
    return INSTANCE;
  }

  @Override
  public PCollection<JournalRecord<JsonNode>> expand(PCollection<String> input) {
    return input.apply(
        FlatMapElements
            .into(new TypeDescriptor<JournalRecord<JsonNode>>() {})
            .via(JsonToJournalRecords::parse));
  }

  private static ImmutableList<JournalRecord<JsonNode>> parse(final String record) {
    try {
      return ImmutableList.copyOf(OBJECT_READER.<JournalRecord<JsonNode>[]>readValue(record));
    } catch (IOException e) {
      throw new RuntimeException(
          String.format("Unable to parse input record as an %s or array", JsonNode.class.getName()),
          e);
    }
  }
}
```

## Use AutoValue and JSON annotation to create builder class
```java
@AutoValue
@JsonSerialize(as = JournalRecord.class)
public abstract class JournalRecord<T> implements Serializable {
  @JsonCreator
  public static <T> JournalRecord<T> of(
      @JsonProperty("json") final T message,
      @JsonProperty("clock") final String clock,
      @JsonProperty("updatedAt") final Instant updatedAt,
      @JsonProperty("status") final String status) {
    // work around https://github.com/FasterXML/jackson-databind/issues/921
    // by removing @JsonDeserialize(builder = AutoValue_ProtoECommEntry.Builder.class)
    // from the class and adding this JsonCreator instead...

    return new AutoValue_JournalRecord.Builder<T>()
        .setMessage(message)
        .setClock(clock)
        .setUpdatedAt(updatedAt)
        .setStatus(status)
        .build();
  }

  @AutoValue.Builder
  public abstract static class Builder<T> {
    public abstract Builder<T> setMessage(T message);

    public abstract Builder<T> setClock(String clock);

    public abstract Builder<T> setUpdatedAt(Instant updatedAt);

    public abstract Builder<T> setStatus(String status);

    public abstract JournalRecord<T> build();
  }

  @JsonProperty("json")
  public abstract T getMessage();

  @JsonProperty("clock")
  public abstract String getClock();

  @JsonProperty("updatedAt")
  public abstract Instant getUpdatedAt();

  @JsonProperty("status")
  public abstract String getStatus();
}
```

## Read from Kinesis

```java
KinesisIO
        .read()
        .withPollingInterval(Duration.millis(1000))
        .withStreamName(kinesisStreamName)
        .withInitialPositionInStream(InitialPositionInStream.TRIM_HORIZON)
        .withAWSClientsProvider(KinesisClientsProvider.of(runMode))
        .withArrivalTimeWatermarkPolicy();
```
