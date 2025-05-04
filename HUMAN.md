# 人間関数のマーメイド図

```mermaid
flowchart TD
    %% 入力情報
    subgraph Input
        subgraph 外部的
            環境
            subgraph 情報
                SNS
            end
    
        end
        subgraph 内部的
            思考結果
        end
    end
  
    Input --> HUMAN
    環境 --> 思考
    情報 --> 思考
    %% 人間関数の中身
    subgraph HUMAN["人間関数: HUMAN(x)"]
        subgraph 思考
        end
    end

    HUMAN --> OUTPUT

    %% 出力
    subgraph OUTPUT
        subgraph 行動
        end
    end
