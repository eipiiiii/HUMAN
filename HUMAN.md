# 人間関数のマーメイド図

```mermaid
flowchart TD
    %% 入力情報
    subgraph Input
        subgraph 外部的
            環境
            subgraph 感覚的入力
                SNS
            end
    
        end
        subgraph 内部的
            subgraph 認知的入力
                記憶
            
            end
        end
        subgraph 社会的/文化的データ
            subgraph 社会的入力
                対人コミュニケーション
                メディア
            end

        end

    end
  
    Input --> HUMAN
    環境 --> 思考
    感覚的入力 --> 思考

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
