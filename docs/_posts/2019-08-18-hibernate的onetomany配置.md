---
title: hibernate onetomany onetoone
date: 2019-08-18 22:32:15
tags: Java Framework
---
```
@Data
@Entity
@Slf4j
@JsonIgnoreProperties({ "handler","hibernateLazyInitializer" })
@Table(name = "currentprice")
public class CurrentPrice {
    @Id
    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private float price;
    private float high;
    private float low;
    private float open;
    private float close;
    private float changed;
    @JsonFormat(pattern = "YYYY-MM-dd", timezone = "GMT+8")
    Date date;

}

@Data
@Entity
@Slf4j
@JsonIgnoreProperties({ "handler","hibernateLazyInitializer" })
@Table(name = "fundmanager")
public class FundManager {
    @Id
    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String name;
    private String password;
    private float balance;
    private float profit;
    @Column(name = "instruments_value")
    private float instrumentsValue;

}

@Data
@Entity
@Slf4j
@JsonIgnoreProperties({ "handler","hibernateLazyInitializer" })
@Table(name = "historyprice")
public class HistoryPrice {
    @Id
    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private float price;
    private float high;
    private float low;
    private float open;
    private float close;
    private float changed;
    @JsonFormat(pattern = "YYYY-MM-dd", timezone = "GMT+8")
    private Date date;

    @ManyToOne(targetEntity = Instrument.class, cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "history_price_instrument", referencedColumnName = "id")
    private Instrument instrument;

}

@Data
@Entity
@Slf4j
@JsonIgnoreProperties({ "handler","hibernateLazyInitializer" })
@Table(name = "hold")
public class Hold {
    @Id
    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private int volume;

    @ManyToOne(targetEntity = Instrument.class, fetch = FetchType.EAGER, cascade = CascadeType.MERGE)
    @JoinColumn(name = "hold_instrument_id", referencedColumnName = "id")
    private Instrument instrument;

    @ManyToOne(targetEntity = FundManager.class, cascade = CascadeType.MERGE, fetch = FetchType.LAZY)
    @JoinColumn(name = "hold_fund_manager_id", referencedColumnName = "id")
    public FundManager fundManager;
}

@Data
@Entity
@Slf4j
@JsonIgnoreProperties({ "handler","hibernateLazyInitializer" })
@Table(name = "instrument")
public class Instrument {
    @Id
    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String symbol;
    private String name;
    @Enumerated(EnumType.STRING)
    private InstrumentType type;


    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "currentprice_id", referencedColumnName = "id")
    private CurrentPrice currentPrice;
}

public enum InstrumentType {
    Bonds("Bonds",1),
    Stocks("Stocks",2),
    Futures("Futures",3),
    ETFs("ETFs",4);

    //instrument Name
    String type;

    //instrument index
    int index;

    public String getType(){
        return type;
    }
    public int getIndex(){
        return index;
    }

    InstrumentType(String type,int index) {
        this.type = type;
        this.index = index;
    }

    public void setType(String type) {
        this.type = type;
    }

    public void setIndex(int index) {
        this.index = index;
    }
}


@Data
@Entity
@Slf4j
@JsonIgnoreProperties({ "handler","hibernateLazyInitializer" })
@Table(name = "tradinghistory")
public class TradingHistory {
    @Id
    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private float price;
    private int volume;
    @JsonFormat(pattern = "YYYY-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date date;

    @ManyToOne(targetEntity = Instrument.class, fetch = FetchType.LAZY, cascade = CascadeType.MERGE)
    @JoinColumn(name = "history_instrument_id", referencedColumnName = "id")
    private Instrument instrument;

    @ManyToOne(targetEntity = FundManager.class, cascade = CascadeType.MERGE, fetch = FetchType.LAZY)
    @JoinColumn(name = "history_fund_manager_id", referencedColumnName = "id")
    public FundManager fundManager;

}

```
