# React跨组件调用

## 父组件

~~~react
import React, { useState, useRef } from 'react';
import './index.scss';
import Students from './components/RightInfoWarn/index'
import Search from '../../assets/consult/search.png';
import InfoA from './components/LeftInfo/InfoA'
import InfoB from './components/LeftInfo/InfoB'
import InfoC from './components/LeftInfo/InfoC'

const StudentForm = () => {
  {/* 子组件*/}
  const infoB = useRef();
	{/* 子组件调用父组件的方法 */}
  const callChild = (userId) => {
    console.log("qwaaqwe"+userId);
    if (infoB.current) {
        infoB.current.someMethod(userId);
    }
  };

  return (
    <div className='big-box'>
      <div className="left-info">
        <InfoA/>
        {/* 子组件名字 */}
        <InfoB ref={infoB} />
        <InfoC/>
      </div>
      <div className='right-box'>
        <div className='lbottom-box'>
          { /* 子组件的的方法 */ }
          <Students triggerSecondChildMethod={callChild}/>
        </div>
      </div>
    </div>
  );
};

export default StudentForm;
~~~

## 子组件A

~~~react
import React, { useRef } from 'react';
import ComparisonCard from './components/ComparisonCard'; // 替换为实际路径

const ParentComponent = () => {
    const comparisonCardRef = useRef();

    const callChild = (userId) => {
        if (comparisonCardRef.current) {
            comparisonCardRef.current.someMethod(userId);
        }
    };

    return (
        <div>
            <button onClick={() => callChild('user123')}>调用子组件方法</button>
            <ComparisonCard ref={comparisonCardRef} />
        </div>
    );
};

export default ParentComponent;

~~~

## 子组件B

~~~java
import React, { useState, forwardRef, useImperativeHandle } from 'react';
import './index.scss';
import documentIcon from '../../../../../assets/consult/info.png'; // 替换为实际路径
import { getUserRecentTwoPerma } from '../../../../../assets/api/aIConsulting'; // Modify the path to your API file

const ComparisonCard = forwardRef((props, ref) => {
    const [selectedPeriod, setSelectedPeriod] = useState('一月前');
    const [previousMetrics, setPreviousMetrics] = useState(null);
    const [currentMetrics, setCurrentMetrics] = useState(null);
    const [improvementRate, setImprovementRate] = useState(null);
	
    const handlePeriodChange = (e) => {
        setSelectedPeriod(e.target.value);
    };

    { /* 被调用的方法 */ }
    useImperativeHandle(ref, () => ({
        someMethod(userId) {
            getUserRecentTwoPerma(userId)
                .then(response => {
                    const data = response.data.data;
 
                    console.log(data,"数据");
                    if (true) {
                        const [latest, previous] = data.perma;
                 
                        console.log(latest,"latest")
                        console.log(previous,"previous")

                        //新用户只有一天的情况
                        if(previous != null){
                            setCurrentMetrics({
                                metrics: [
                                    { label: '积极', score: previous.permaP },
                                    { label: '人际', score: previous.permaR },
                                    { label: '投入', score: previous.permaE },
                                    { label: '意义', score: previous.permaM },
                                    { label: '成就', score: previous.permaA }
                                ]
                            });
                        }else{
                            //setCurrentMetrics
                            //setPreviousMetrics
                            setCurrentMetrics({
                                metrics: [
                                    { label: '积极', score: null },
                                    { label: '人际', score: null },
                                    { label: '投入', score: null },
                                    { label: '意义', score: null },
                                    { label: '成就', score: null }
                                ]
                            });
                        }

                        if(latest != null){
                            setPreviousMetrics({
                                metrics: [
                                    { label: '积极', score: latest.permaP },
                                    { label: '人际', score: latest.permaR },
                                    { label: '投入', score: latest.permaE },
                                    { label: '意义', score: latest.permaM },
                                    { label: '成就', score: latest.permaA }
                                ]
                            });
                        }else{
                            setPreviousMetrics({
                                metrics: [
                                    { label: '积极', score: null },
                                    { label: '人际', score: null },
                                    { label: '投入', score: null },
                                    { label: '意义', score: null },
                                    { label: '成就', score: null }
                                ]
                            });
                        }

                        setImprovementRate((data.impValue * 100).toFixed(2));
                    } else {
                        console.error('Unexpected data format:', data);
                    }
                })
                .catch(error => {
                    console.error('There was an error fetching the data!', error);
                });
        }
    }));

    return (
        <div className="comparison-card">
            <div className="header">
                <img src={documentIcon} alt="Document Icon" className="icon" />
                <span className="title">与昨天的对比</span>
                <select className="period-select" value={selectedPeriod} onChange={handlePeriodChange}>
                    <option value="一月前">对比</option>
                </select>
            </div>
            <div className="body">
                {previousMetrics && (
                    <div className="metrics-section">
                        <div className="total-score previous">{previousMetrics.totalScore}</div>
                        <div className="metrics">
                            {previousMetrics.metrics.map((metric, index) => (
                                <div className="metric" key={index}>
                                    <span className="metric-label">{metric.label} {metric.score}</span>
                                    <div className="progress-bar">
                                        <div className="progress previous" style={{ width: `${metric.score}%` }}></div>
                                    </div>
                                </div>
                            ))}
                        </div>
                    </div>
                )}
                {currentMetrics && (
                    <div className="metrics-section">
                        <div className="total-score current">{currentMetrics.totalScore}</div>
                        <div className="metrics">
                            {currentMetrics.metrics.map((metric, index) => (
                                <div className="metric" key={index}>
                                    <span className="metric-label">{metric.label} {metric.score}</span>
                                    <div className="progress-bar">
                                        <div className="progress current" style={{ width: `${metric.score}%` }}></div>
                                    </div>
                                </div>
                            ))}
                        </div>
                    </div>
                )}
            </div>
            <div className="footer">
                {improvementRate !== null && (
                    <span className="improvement-rate">
                        <span className="dot"></span>
                        改善率达 {improvementRate}%
                    </span>
                )}
            </div>
        </div>
    );
});

export default ComparisonCard;

~~~

